# 병렬 데이터 처리와 성능
## 병렬 스트림
* 컬렉션에 parallelStream을 호출하면 병렬 스트림이 생성
    * 병렬 스트림 : 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림
* 병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당 가능

### 순차 스트림을 병렬 스트리으로 변환하기
```java
public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
                 .limit(n)
                 .parallel()    // 스트림을 병렬 스트림으로 변환
                 .reduce(0L, Long::sum)
}
```
* 스트림이 여러 청크로 분할되어 있음
* 리듀싱 연산을 여러 청크에 병렬로 수행 가능
* 리듀싱 연산으로 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐서 전체 스트림의 리듀싱 결과를 도출

```java
stream.parallel()
      .filter(...)
      .sequential()
      .map(...)
      .parallel()       // 마지막 호출이 parallel 이므로 파이프라인은 전체적으로 병렬로 실행
      .reduce();
```
* parallel을 호출하면 이후 연산이 병렬로 수행해야 함을 의미하는 불리언 플래그가 설정
* sequential은 병렬 스트림을 순차 스트림으로 변환
* parallel과 sequential 두 메서드 중 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미침

### 스트림 성능 측정
* 반복형, 순차 리듀싱, 병렬 리듀싱 성능 비교
    * JMH(자바 마이크로벤치마크 하니스) 라이브러리 사용
* 반복형 테스크 코드
    ```java
    @Benchmark
    public long iterativeSum() {
        long result = 0;
        for (long i = 1L; i <= N; i++) {
            return += i;
        }
        return result;
    }
    ```
* 순차 리듀싱 테스트 코드
    ```java
    @Benchmark
    public long sequentialSum() {
        return Stream.iterate(1L, i -> i + 1)
                     .limit(N)
                     .reduce(0L, Long::sum);
    }
    ```
* 병렬 리듀싱 테스트 코드
    ```java
    @Benchmark
    public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
                 .limit(n)
                 .parallel()
                 .reduce(0L, Long::sum)
    }
    ```
* 전통적인 for 루프를 사용해 반복하는 방법이 더 저수준으로 동작할 뿐 아니라 특히 기본값을 박싱하거나 언박싱할 필요가 없으므로 순차적 스트림을 사용하는 버전에 비해 거의 4배 이상 빠른 결과가 나옴.
* 병렬 버전이 쿼드 코어 CPU를 활용하지 못하고 순차 버전에 비해 다섯배나 느린 결과가 나옴.
    * 두 가지 문제점
        * 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야 함
        * 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기가 어려움
            * `이전 연산의 결과에 따라 다음 함수의 입력이 달라지기 때문에 iterate 연산을 청크로 분할하기 어려움`

#### 더 특화된 메서드 사용
* iterate -> LongStream.rangeClosed 사용
    * LongStream.rangeClosed는 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라짐
    * LongStream.rangeClosed는 쉽게 청크로 분할할 수 있는 숫자 범위를 생산
        * 예를 들어 1-20 범위의 숫자를 각각 1-5,6-10,11-15,16-20 범위의 숫자로 분할
* 변경된 순차 리듀싱 테스트 코드
    ```java
    @Benchmark
    public long sequentialSum() {
        return Stream.rangeClosed(1, N)
                     .reduce(0L, Long::sum);
    }
    ```
    * 기존 순차 리듀싱 테스크 코드(iterate 사용)보다 처리속도가 빠름.
        * `상황에 따라서는 어떤 알고리즘을 병렬화하는 것보다 적적한 자료구조를 선택하는 것이 더 중요`
* 변경된 병렬 리듀싱 테스트 코드
    ```java
    @Benchmark
    public long sequentialSum() {
        return Stream.rangeClosed(1, N)
                     .parallel()
                     .reduce(0L, Long::sum);
    }
    ```
    * 순차 실행보다 빠른 성능
    * 올바른 자료구조를 선택해야 병렬 실행도 최적의 성능을 발휘할 수 있다는 사실을 확인할 수 있음
* 병렬화의 비용은 비쌈
    * 코어 간에 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 다른 코어에서 수행하는 것이 바람직

### 병렬 스트림의 올바른 사용법
* 병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 발생
    ```java
    public long sideEffectParallelSum(long n) {
        Accumulator accumulator = new Accumulator();
        LongStream.rageClosed(1, n)
                  .parallel()
                  .forEach(accumulator::add);
        return accumulator.total;
    }

    public class Accumulator {
        public long total = 0;
        public void add(long value) {
            total += value;
        }
    }
    ```
    * total 접근시 (다수의 스레드에서 동시에 데이터에 접근하는) 데이터 레이스 문제가 발생
    * 병렬 스트림과 병렬 계산에서는 공유된 가변 상태를 피해야 함

### 병렬 스트림 효과적으로 사용하기
* 순차 스트림과 병렬 스트림 중 어떤 것이 좋을지 모르겠다면 적절한 벤치마크로 직접 성능을 측정하는 것이 바람직함
* 박싱을 주의
    * 되도록이면 기본형 특화 스트림을 사용하는 것이 좋음
* 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있음
    * limit나 findFirst처럼 요소의 순서에 의존하는 연산을 병렬 스트림에서 수행하려면 비싼 비용을 치뤄야 함
* 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려해야 함
    * N(처리해야 할 요소 수) * Q(하나의 요소를 처리하는 데 드는 비용)
    * Q가 높아진다는 것은 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있음을 의미
* 소량의 데이터에서는 병렬 스트림이 도움되지 않음
* 스트림을 구성하는 자료구조가 적절한지 확인
    * ArrayList를 LinkedList보다 효율적으로 분할 할 수 있음
        * LinedList를 분할하려면 모든 요소를 탐색해야 하지만 ArrayList는 요소를 탐색하지 않고도 리스트를 분할 할 수 있기 때문
* 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있음
    * SIZED 스트림은 정확한 같은 크기의 두 스트림으로 분할 할 수 있음으로 병렬 처리가 효과적
    * 필터 연산이 있으면 스트림의 길이를 예측할 수 없으므로 병렬 처리가 효과적일지는 미지수
* 최종 연산의 병합과정 비용을 살펴봐야 함
    * 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻은 성능의 이익이 서브트림의 부분 결과를 합치는 과정에서 상쇄될 수 있음

#### 스트림 소스와 분해성
|소스|분해성|
|---|---|
|ArrayList|훌륭함|
|LinkedList|나쁨|
|IntStream.range|훌륭함|
|Stream.iterate|나쁨|
|HashSet|좋음|
|TreeSet|좋음|

## 포크/조인 프레임워크
* 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계
* 서브 태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현

### RecursiveTask 활용
* 스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야 함
    * R은 병렬화된 태스크가 생성하는 결과 형식 또는 결과가 없을 때는 RecursiveAction 형식
    * RecursiveTask를 정의하려면 추상 메서드 compute를 구현해야 함
        ```java
        protected abstrat R compute();
        ```
    * compute 메서드 구현의 의사코드 형식
        ```java
        if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
            순차적으로 태스트 계산
        } else {
            태스크를 두 서브태스크로 분할
            태스크가 다시 서브 태스크로 분할되도록 이 메서드를 재귀적으로 호출함
            모든 서브태스크의 연산이 완료될 때까지 기다림
            각 서브태스크의 결과를 합침
        }
        ```
* 포크/조인 프레임워크를 이용해서 병렬 합계 수행하는 예제
    ```java
    public class ForkJoinSumCalculator extends RecursiveTask<Long> {    // RecursiveTask를 상속받아 포크/조인 프레임워크에서 사용할 태스크를 생성

        public static final long THRESHOLD = 10_000;    // 이 값 이하의 서브태스크는 더 이상 분할 할 수 없음.

        private final long[] numbers;   // 더할 숫자 배열
        private final int start;        // 이 서브태스크에서 처리할 배열의 초기 위치
        private final int end;          // 최종 위치

        // 메인 태스크를 생성할 때 사용할 공개 생성자
        public ForkJoinSumCalculator(long[] numbers) {
            this(numbers, 0, numbers.length);
        }

        // 메인 태스크의 서브태스크를 재귀적으로 만들 때 사용할 비공개 생성자
        private ForkJoinSumCalculator(long[] numbers, int start, int end) {
            this.numbers = numbers;
            this.start = start;
            this.end = end;
        }

        // RecursiveTask의 추상메서드 오버라이드
        @Override
        protected Long compute() {
            int length = end - start;       // 이 태스크에서 더할 배열의 길이

            // 기준값과 같거나 작으면 순차적으로 결과를 계산
            if (length <= THRESHOLD) {
                return computeSequentially();
            }

            // 배열의 첫 번째 절반을 더하도록 서브태스크를 생성
            ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length / 2);
            leftTask.fork();    // ForkJoinPool의 다른 스레드로 새로 생성한 태스크를 비동기로 실행

            // 배열의 나머지 절반을 더하도록 서브태스크를 생성
            ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length / 2, end);
            Long rightResult = rightTask.compute();     // 두 번째 서브태스크를 동기 실행. 이때 추가로 분할이 일어날 수 있음
            Long leftResult = leftTask.join();          // 첫 번째 서브태스크의 결과를 읽거나 아직 결과가 없으면 기다림

            return leftResult + rightResult;            // 두 서브태스크의 결과를 조합한 값이 이 태스크의 결과
        }

        // 더 분할 할 수 없을 때 서브태스크의 결과를 계산하는 단순한 알고리즘
        private long computeSequentially() {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += numbers[i];
            }
            return sum;
        }

        public static long forkJoinSum(long n) {
            long[] numbers = LongStream.rangeClosed(1, n).toArray();
            ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
            return FORK_JOIN_POOL.invoke(task);
        }

    }
    ```
    * 병렬 스트림을 이용할 때보다 성능이 나빠짐.
        * ForkJoinSumCalculator 태스크에서 사용할 수 있도록 전체 스트림을 long[]으로 변환했기 때문

#### ForkJoinSumCalculator 실행
* ForkJoinSumCalculator를 ForkJoinPool로 전달하면 풀의 스레드가 ForkJoinSumValvulator의 compute 메서드를 실행하면서 작업 수행
* compute 메서드는 병렬로 실행 할 수 있을 만큼 태스크의 크가가 충분히 작아졌는지 확인
* 아직 태스크의 크기가 크다고 판단되면 숫자 배열을 반으로 배열해서 두 개의 새로운 ForkJoinSumCalculator로 할당
* 다시 ForkJoinPool이 새로 생성된 ForkJoinSumCalculator를 실행
* 위의 과정이 재귀적으로 반복되면서 주어진 조건을 만족할 때까지 태스크 분할을 반복
* 각 서브태스크는 순차적으로 처리되며 포킹 프로세스로 만들어진 이진트리의 태스크를 루트에서 역순으로 방문. 즉, 각 서브태스크의 부분 결과를 합쳐서 태스크의 최종 결과를 계산

### 포크/조인 프레임워크를 제대로 사용하는 방법
* join 메서드를 태스크에서 호출하면 태스크가 생상하는 결과가 준비될 때까지 호출자를 블록
    * 두 서브태스크가 모두 시작된 다음에 join을 호출해야 함
* RecursiveTask 내에서는 ForkJoinPool의 invoke 메서드를 사용하지 말아야 함
    * 대신, compute나 fork 메서드를 직접 호출
    * 순차 코드에서 병렬 계산을 시작할 때만 invode 사용
* 서브 태스크에서 fork 메서드를 호출해서 ForkJoinPool의 일정을 조절할 수 있음
    * 왼쪽 작업과 오른쪽 작업 모두에 fork 메서드를 호출하는 것이 나아보이지만 한쪽 작업에는 fork를 호출하는 것보다는 compute를 호출하는 것이 효율적
    * 두 서브 태스크의 한 태스크에는 같은 스레드를 재사용할 수 있으므로 풀에서 불필요한 태스크를 할당하는 오버헤드를 피할 수 있음
* 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅 하기 어려움
    * 포크/조인 프레임워크에서는 fork라 불리는 다른 스레드에서 compute를 호출하므로 어려움
* 멀티 코어에서 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠를 거라는 생각은 버려야 함
    * 병렬 처리로 성능을 개선하려면 태스크를 여러 독립적인 서브태스크로 분할 할 수 있어야 함
    * 각 서브태스크의 실행시간은 새로운 태스크를 포킹하는데 드는 시간보다 길어야 함

### 작업 훔치기
* 코어 개수와 관계없이 적잘한 크리로 분할된 많은 태스크를 포킹하는 것이 바람직
* 현실예선 각가의 서브 태스크의 작업완료 시간이 크게 달라질 수 있음
    * `작업 훔치기(work stealling)` 기법으로 문제를 해결 할 수 있음
        * 작업 훔치기 기법에서는 ForkJoinPool의 모든 스레드를 거의 공정하게 분할
        * 다른 스레드는 바쁘게 일하고 있는데 한 스레드는 할일이 다 떨어진 상황이라면 다른 스레드 큐 꼬리에서 작업을 훔쳐옴.
        * 태스크의 크기를 작게 나누어야 작업자 스레드 간의 작업 부하를 비슷한 수준으로 유지 할 수 있음

## Spliterator 인터페이스
> 자동으로 스트림을 분할하는 기법 제공
* Spliterator 병렬 작업에 특화됨
* Spliterator 인터페이스
    ```java
    public interface Spliterator<T> {
        boolean tryAdvance(Consumer<? super T> action;
        Spliterator<T> trySplit();
        long estimateSize();
        int characteristics();
    }
    ```
    * T는 Spliterator에서 탐색하는 요소의 형식을 가리킴
    * tryAdvance 메서드는 Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 참을 반환
    * trySplit 메서드는 Spliterator의 일부 요소를 분할해서 두 번째 Spliterator를 생성하는 메서드
    * Spliterator에서는 estimateSize 메서드로 탐색해야 할 요소 수 정보를 제공
        * 탐색해야 할 요소 수가 정확하진 않더라도 제공된 값을 이용해서 더 쉽고 공평하게 Spliterator를 분할 할 수 있음

### 분할 과정
* 1단계 : 첫 번째 Spliterator에 trySplit을 호출하면 두 번째 Spliterator가 생성됨
* 2단계 : 두 개의 Spliterator에 trySplit를 다시 호출하면 네 개의 Spliterator가 생성. trySplit의 결과가 null이 될 때까지 이 과정을 반복
* 3단계 : trySplit이 null을 반환했다는 것은 더 이상 자료구조를 분할 할 수 없음을 의미
* 4단계 : Spliterator에 호출한 모든 trySplit의 결과가 null이면 재귀 분할 과정이 종료

#### Spliterator 특성
* Spliterator는 characteristics라는 추상 메서드도 정의
    * Characteristics 메서드는 Spliterator 자체의 특성 집합을 포함하는 int를 반환
* Spliterator 특성
    |특성|의미|
    |---|---|
    |ORDERED|리스트처럼 요소에 정해진 순서가 있으므로 Spliterator는 요소를 탐색하고 분할할 때 이 순서에 유의해야 함|
    |DISTINCT|x,y 두 요소를 방문했을 때 x.equals(y)는 항상 false를 반환|
    |SORTED|탐색된 요소는 미리 정의된 정렬 순서를 따름|
    |SIZED|크기가 알려진 소스로 Spliterator를 생성햇으므로 estimatedSize()는 정확한 값을 반환|
    |NON-NULL|탐색하는 모든 요소는 null이 아님|
    |IMMUTABLE|이 Spliterator의 소스는 불변. 즉 요소를 탐색하는 동안 요소를 추가하거나 삭제하거나 고칠 수 없음|
    |CONCURRENT|동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있음|
    |SUBSIZED|이 Spliterator 그리고 불할되는 모든 Spliterator는 SIZED 특성을 갖음|

### 커스텀 Spliterator 구현하기
* 반복형으로 단어 수를 세는 메서드
    ```java
    public static int countWordsIteratively(String s) {
        int counter = 0;
        boolean lastSpace = true;
        for (char c : s.toCharArray()) {        // 문자열의 모든 문자를 하나씩 탐색
            if (Character.isWhitespace(c)) {
                lastSpace = true;
            } else {
                if (lastSpace) {
                    counter++;        // 문자를 하나씩 탐색하다가 공백 문자를 만나면 지금까지 탐색한 문자를 단어로 간주하여(공백 문자는 제외) 단어 수를 증가
                }
                lastSpace = Character.isWhitespace(c);
            }
        }
        return counter;
    }
    ```
#### 함수형으로 단어 수를 세는 메서드 재구현하기
* 문자열 스트림을 탐색하면서 단어 수를 세는 클래스
    ```java
    private static class WordCounter {

        private final int counter;
        private final boolean lastSpace;

        public WordCounter(int counter, boolean lastSpace) {
            this.counter = counter;
            this.lastSpace = lastSpace;
        }

        // 반복 알고리즘처럼 accumulate 메서드는 문자열의 문자를 하나씩 탐색
        public WordCounter accumulate(Character c) {
            if (Character.isWhitespace(c)) {
                return lastSpace ? this : new WordCounter(counter, true);
            } else {
                return lastSpace ? new WordCounter(counter + 1, false) : this;      // 문자열 하나씩 탐색하다 공백 문자를 만나면 지금까지 탐색한 문자를 단어로 간주하여(공백문자는 제외) 단어 수를 증가시킴
            }
        }

        public WordCounter combine(WordCounter wordCounter) {
            // 두 WordCounter의 counter 값을 더함, counter 값만 더할것이므로 마지막 공백은 신경 쓰지 않음
            return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
        }

        public int getCounter() {
            return counter;
        }

    }
    ```

#### WordCounter 병렬로 수행하기
* 순차 스트림을 병렬 스트림으로 바꿀 때 스트림 분할 위치에 따라 잘못된 결과가 나올 수 있음
    * 문자열을 임의의 위치에서 분할하지 말고 단어가 끝나는 위치에서만 분할하는 방법으로 문제를 해결할 수 있음
        ```java
        private static class WordCounterSpliterator implements Spliterator<Character> {

            private final String string;
            private int currentChar = 0;

            private WordCounterSpliterator(String string) {
                this.string = string;
            }

            @Override
            public boolean tryAdvance(Consumer<? super Character> action) {
                action.accept(string.charAt(currentChar++));        // 형재 문자를 소비
                return currentChar < string.length();       // 소비할 문자가 남아있으면 true 반환
            }

            @Override
            public Spliterator<Character> trySplit() {
                int currentSize = string.length() - currentChar;

                // 파싱할 문자열을 순차 처리할 수 있을 만큼 충분히 작아졌음을 알리는 null을 반환
                if (currentSize < 10) {
                    return null;
                }

                // 파싱할 문자열의 중간을 불할 위치로 설정
                for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
                    // 다음 공백이 나올 때까지 분할 위치를 뒤로 이동 시킴
                    if (Character.isWhitespace(string.charAt(splitPos))) {
                        // 처음부터 분할 위치까지 문자열을 파싱할 새로운 WordCounterSpliterator를 생성
                        Spliterator<Character> spliterator = new WordCounterSpliterator(string.substring(currentChar, splitPos));
                        currentChar = splitPos;     // WordCounterSpliterator를의 시작 위치를 분할 위치로 설정
                        return spliterator;         // 공백을 찾았고 문자열을 분리햇으므로 루프 종료
                    }
                }

                return null;
            }

            // 탐색해야 할 요소의 개수(estimateSize)는 Spliterator가 파싱할 문자열 전체 길이와 현재 반복 중인 위치의 차
            @Override
            public long estimateSize() {
                return string.length() - currentChar;
            }

            // 프레임 워크에서 Spliterator의 특성들을 알림
            @Override
            public int characteristics() {
                return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
            }

        }
        ```

## 마치며
* 내부 반복을 이요하면 명시적으로 다른 스레드를 사용하지 않고도 스트림을 병렬로 처리 할 수 있다.
* 간단하게 스트림을 병렬로 처리할 수 있지만 항상 병렬 처리가 빠른 것은 아니다. 병렬 소프트웨어 동작 방법과 성능은 직관적이지 않을 때가 많으므로 병렬 처리를 사용했을 때 성능을 직접 측정해봐야 한다.
* 병렬 스트림으로 데이터 집합을 병렬 실행할 때 특히 처리해야 할 데이터가 아주 많거나 각 요소를 처리하는 데 오랜 시간이 걸릴 때 성능을 높일 수 있다.
* 가능하면 기본형 특화 스트림을 사용하는 등 올바른 자료구조 선택이 어떤 연산을 병렬로 처리하는 것보다 성능적으로 더 큰 영향을 미칠 수 있다.
* 포크/조인 프레임워크에서는 병렬화할 수 있는 태스크를 작은 태스크로 분할한 다음에 분할된 태스크를 각각의 스레드를 실행하며 서브태스크의 각각의 결과를 합쳐서 최종 결과를 생산한다.
* Spliterator는 탐색하려는 데이터를 포함하는 스트림을 어떻게 병렬화할 것인지 정의한다.