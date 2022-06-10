# 람다 표현식
## 람다란 무엇인가?
> 람다 표현식 : 메서드로 전달 할 수 있는 익명 함수를 단순화한 것

* 람다의 특징
    * 익명
        * 보통의 메서드와 달리 이름이 없으므로 익명이라 표현.
    * 함수
        * 람다는 메서드처럼 특정 클래스에 종속되지 않음
        * 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함
    * 전달
        * 람다 표현식을 메서드 인수로 전달하거나 변수로 저장
    * 간결성
        * 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없음
* 람다 표현식
    ```java
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
    ```
    * 파라미터 리스트
        * (Apple a1, Apple a2)
        * Comparator의 compare 메서드 파라미터(사과 두개)
    * 화살표
        * -> 는 람다의 파라미터 리스트와 바디를 구분
    * 람다 바디
        * a1.getWeight().compareTo(a2.getWeight());
        * 람다의 반환값에 해당하는 표현식

* 람다 예제
    |사용 사례|람다 예제|
    |------|---|
    |불리언 표현식|List<String> list -> list.isEmpty()|
    |객체 생성|() -> new Apple(10)|
    |객체에서 소비|(Apple a) -> { Systme.out.println(a.getWeight()); }|
    |객체에서 선택/추출|(String s) -> s.length() |
    |두 값을 조합|(int a, int b) -> a * b|
    |두 객체 비교|(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());|

## 어디에, 어떻게 람다를 사용할까?
### 함수형 인터페이스
> 함수형 인터페이스 : 정확히 하나의 추상 메서드를 지정하는 인터페이스
* 함수형 인터페이스들
    ```java
    // java.util.Comparator
    public interface Comparator<T> {
        int compare(T o1, T o2);
    }

    // java.lang.Runnable
    public interface Runnable {
        void run();
    }

    // 등등..
    ```
    * 많은 디폴트 메서드가 있더라도 `추상 메서드가 오직 하나`면 함수형 인터페이스임

* 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있음
    ```java
    // 람다 사용
    Runnable r1 = () -> System.out.println("Hello World 1");

    // 익명 클래스 사용
    Runnable r2 = new Runnable() {
        public void run() {
            System.out.println("Hello World 2");
        }
    }

    public static void process(Runnable r) {
        r.run();
    }

    process(r1);
    process(r2);
    // 직접 전달된 람다 표현식
    process(() -> System.out.println("Hello World 3"));
    ```

### 함수 디스크립터
> 함수 디스크립터 : 람다 표현식의 시그니처를 서술하는 메서드
* 람다 표현식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다
    ```java
    public void process(Runnable r) {
        r.run();
    }

    process(() -> System.out.println("This is awesome!!"));
    ```
    * 인수가 없으며 void를 반환하는 람다 표현식. 이는 Runnable 인터페이스의 run 메서드 시그니처와 같음

* @FunctionallInterface
    * 함수형 인터페이스임을 가리키는 어노테이션
    * 선언했지만 실제로 함수형 인터페이스가 아니라면 컴파일 에러 발생

## 람다 활용 : 실행 어라운드 패턴
> 실행 어라운드 패턴 : 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태

* 기존 코드
```java
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();
    }
}
```

### 1단계 : 동작 파라미터화를 기억하라
* processFile의 동작을 파라미터화
    ```java
    String result = processFile(BufferedReader br) -> br.readLine());
    ```

### 2단계 : 함수형 인터페이스를 이용해서 동작 전달
* 함수형 인터페이스 자리에 람다를 사용할 수 있음. BufferedReader -> String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 생성
    ```java
    @FuntionalInterface
    public interface BufferedReaderProcessor {
        String process(BufferedReader b) throws IOException;
    }

    // 정의한 인터페이스를 processFile 메서드의 인수로 전달 할 수 있음
    public String processFile(BufferedReaderProcessor p) throws IOException {
        ...
    }
    ```

### 3단계 : 동작 실행
* processFile 바디 내에서 BufferedReaderProcessor 객체의 process를 호출 할 수 있음
    ```java
    public String processFile(BufferedReaderProcessor p) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
            return p.process(br);
        }
    }
    ```

### 4단계 : 람다 전달
* 람다를 이용해서 다양한 동작을 processFile 메서드로 전달
    ```java
    String oneLine = processFile((BufferedReader br) -> br.readLine());
    ```
