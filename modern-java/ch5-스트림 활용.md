# 스트림 활용
## 필터링
### 프레디케이트로 필터링
* filter 메서드는 프레디케이트(불리언으로 반환하는 함수)를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환.

### 고유 요소 필터링
* distinct 메서드도 지원
    * 중복된 요소 필터링

## 스트림 슬라이싱
### 프레디케이트를 이용한 슬라이싱
#### TAKEWIHLE 활용
* takeWhile을 이용하면 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스 할 수 있음
    ```java
    // 320 보다 크거나 같은 요리가 나올 경우 반복 작업 중간.
    // 조건이 거짓이 될 경우 작업한 결과를 내뱉고 중단
    List<Dish> sliceMenu1 = specialMenu.stream()
                                       .takeWhile(dish -> dis.getCalories() < 320)
                                       .collect(toList());
    ```

#### DROPWHILE 활용
* 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버림. takeWhile과는 정반대
    ```java
    // 320 보다 크거나 같은 요리가 나올 경우 반복 작업 중간.
    // 조건이 거짓이 될 경우 남은 결과를 내뱉고 중단.
    List<Dish> sliceMenu1 = specialMenu.stream()
                                       .dropWhile(dish -> dis.getCalories() < 320)
                                       .collect(toList());
    ```
### 스트림 축소
* limit(n)
    * 최대 요소 n개 반환

### 요소 건너뛰기
* skip(n)
    * 처음 요소 n개를 제외한 스트림을 반환

## 매핑
### 스트림의 각 요소에 함수 적용하기
* map
    * 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑
    ```java
    List<String> dishNames = menu.stream()
                                 .map(Dish::getName)    // Stream<String>
                                 .collect(toList());
    ```
### 스트림 평면화
* 리스트에서 고유 문자로 이루어진 리스트를 반환하고 싶다면?
    * ["Hello", "World"]  ->  ["H","e","l","o","W","r","d"]
        * 앞서 배운 map을 사용해본다면?
        ```java
        worlds.stream()
              .map(word -> word.split(""))      // Stream<String[]>
              .distinct()                       // Stream<String[]>
              .collect(toList());               // List<String[]>
        ```
        * 원하는 결과로 나오지 않음

#### flatMap 사용
```java
List<String> uniqueCharacters = worlds.stream()
                                      .map(word -> word.split(""))      // Stream<String[]>
                                      .flatMap(Arrays::stream)          // Stream<String>
                                      .distinct()                       // Stream<String>
                                      .collect(toList());               // List<String>
```
* 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행

## 검색과 매칭
* anyMatch, allMatch, noneMatch 세 메세드는 스트림 쇼트서킷 기법, 즉 자바의 &&, ||와 같은 연산을 활용

> 쇼트 서킷 : 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과가 상관없이 전체 결과도 거짓이 되는 상황

### 플레디케이스가 적어도 한 요소와 일치하는 지 확인
* anyMatch
```java
if (menu.stream().anyMatch(Dish::isVegetarian)) {
    System.out.println("The menu is (somewhat) vegetarian friendly!!");
}
```

### 프레디케이트가 모든 요소와 일치하는지 검사
* allMatch

### noneMatch
* 주어진 프레디케이트와 일치하는 요소가 없는지 확인

### 요소 검색
* findAny
    ```java
    Optinal<Dish> dish = menu.stream()
                             .filter(Dish::isVegetarian)
                             .findAny();
    ```
#### Optional 이란?
* Optional<T> 클래스(java.util.Optional)는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스
* Optional은 값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공
    * isPresent()
        * Optional이 값을 포함하면 참(true) 반환, 값을 포함하지 않으면 거짓(false) 반환
    * ifPresent(Consumer<T> block)
        * 값이 있으면 주어진 블록을 실행
    * T get()
        * 값이 존재하면 값을 반환. 없으면 NoSuchElementException 발생
    * T orElse(T other)
        * 값이 있으면 값을 반환. 없으면 기본 값을 반환
    ```java
    menu.stream()
        .filter(Dish::isVegetarian)
        .findAny()          // Optional<Dish> 반환
        .ifPresent(dish -> System.out.println(dish.getName()));     // 값이 있으면 출력되고, 없으면 아무일도 일어나지 않음.
    ```

### 첫 번째 요소 찾기
* findFirst()

#### findFirst와 findAny는 언제 사용하나?
* 병렬 실행에서는 첫 번재 요소를 찾기 어려움
* 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용하는게 나음

## 리듀싱
### 요소의 합
```java
int sum = numbers.stream().reduce(0, (a,b) -> a + b);

// 좀 더 간결하게 바꾼다면
int sum = numbers.stream().reduce(0, Integer::sum);
```
* reduce의 두개의 인자
    * 초기값
        * 초기값을 보내지 않는다면 Optional 객체를 반환
    * 두 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T>

### 최대값과 최소값
* 최대값
    ```java
    Optional<Integer> max = numbers.stream().reduce(0, Integer::max);
    ```
* 최소값
    ```java
    Optional<Integer> min = numbers.stream().reduce(0, Integer::min);
    ```

#### reduce 메서드의 장점과 병렬화
* 반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화하기 어려움
* ```java
    int sum = nembers.parallelStream().resduce(0, Integer::sum);
    ```
    * reduce에 넘겨준 람다의 상태(인스턴스 변수)가 바뀌지 말아야 하며, 연산이 어떤 순서로 실행되더라도 결과가 바뀌지 않는 구조여야 함.

#### 스트림 연산 : 상태 없음과 상태 있음
* map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보냄.
    * 보통 상태가 없는, 즉 `내부 상태를 갖지 않는 연산`
* reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요
    * 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정
* sorted, distinct 같은 연산은 스트림의 요소를 정렬하거나 중복을 제거할려면 과거의 이력을 알고 있어야 함
    * 어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어야 있어야 함
    * 데이터 스트림의 크기가 크거나 무한이라면 문제가 생길 수 있음
    * `내부 상태를 갖는 연산`

## 숫자형 스트림
### 기본형 특화 스트림
* 특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지 않음

#### 숫자 스트림으로 매핑
* mapToInt, mapToDouble, mapToLong
* map 과 정확히 같은 기능을 수행하지만 Stream<T> 대신 특화된 스트림을 반환
    ```java
    int calories = menu.stream()                        // Stream<Dish> 반환
                    .mapToInt(Dish::getCalories)     // IntStream 반환
                    .sum();
    ```
    * IntStream 인터페이스에서 제공하는 sum 메서드를 이용할 수 있음.
    * 스트림이 비어있으면 sum은 기본값 0을 반환
    * IntStream은 max, min, average 등 다양한 유틸리티 메서드도 지원

#### 객체 스트림으로 복원하기
* boxed 메서드
    * 특화 스트림을 일반 스트림으로 변환 할 수 있음
    ```java
    IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
    Stream<Integer> stream = intStream.boxed();
    ```

#### 기본값 : OptionalInt
* IntStream은 기본값 0 때문에 잘못된 결과가 도출 될 수 있음
* 스트림에 요소가 없는 상황과 실제 값이 0인 경우를 어떻게 구별할까?
    ```java
    OptionalInt maxCalories = menu.stream().mapToInt(Dish::getCalories).max();
    int max = maxCalories.orElse(1);    // 값이 없을 때 기본 최대값을 명시적으로 설정
    ```

### 숫자 범위
* range
    * 첫번째 인수로 시작값, 두번째 인수로 종료값
    * 시작과 종료값이 결과에 포함되지 않음
* rangeClosed
    * 첫번째 인수로 시작값, 두번째 인수로 종료값
    * 시작값과 종료값이 결과에 포함됨

## 스트림 만들기
### 값으로 스트림 만들기
* Stream.of : 스트림 생성
    ```java
    Strema<String> stream = Stream.of("Modern", "java", "In", "Action");
    stream.map(String::toUpperCase).forEach(System.out::println);
    ```
* empty 메서드 : 스트림 비움
    ```java
    Stream<String> emptyStream = Stream.empty();
    ```

### null이 될 수 있는 객체로 스트림 만들기
* 때로는 null이 될 수 있는 객체를 스트림으로 만들어야 할 경우가 있음.
    * 가령 예를 들어 System.getProperty는 제공된 키에 대응하는 속성이 없으면 null을 반환
    ```java
    String homeValue = System.getProperty("home");
    Stream<String> homeValueStream  = homeValue == null ? Stream.empty() : Stream.of(value);
    ```
    * Stream.ofNullable 이용
        ```java
        Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
        ```

### 배열로 스트림 만들기
* Arrays.stream
    ```java
    int[] numbers = {2, 3, 5, 7, 11, 13};
    int sum = Arrays.stream(numbers).sum();
    ```

### 파일로 스트림 만들기
* java.nio.file.Files의 많은 정적 메서드가 스트림을 반환
    ```java
    long uniqueWords = 0;
    try (String<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
        uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                           .distinct()
                           .count();
    } catch (IOException e) {
        // 파일을 열다가 예외가 발생하면 처리한다.
    }
    ```
    * Stream 인터페이스는 AutoCloseable 인터페이스를 구현하기 때문에 try 블록 내의 자원은 자동으로 관리됨
    * line에 splict 메서드를 호출해서 각 행의 단어를 분리
    * 각 행의 단어를 여러 스트림으로 만드는 것이 아니라 flatMap으로 스트림을 하나로 평면화

### 함수로 무한 스트림 만들기
* Stream.iterate, Stream.generate
    * `무한 스트림`. 즉, 고정된 컬렉션에서 고정된 크기로 스트림을 만들었던 것과는 달리 크기가 고정되지 않은 스트림을 생성

#### iterate 메서드
```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```
* 초기값과 람다를 인수로 받아서 새로운 값을 끊임없이 생성
* 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 무한 스트림을 만듬
* 이러한 스트림을 언바운드 스트림(unbounded stream)
* 프레디케이트를 지원
    ```java
    // 0에서 시작해서 100보다 크면 숫자 생성을 중단하는 코드
    IntStream.interate(0, n -> n < 100, n -> n + 2)
             .forEach(System.out::println);
    ```
    * takeWhile을 이용하여 작업을 중단 할 수도 있음

#### generate 메서드
* interate와 달리 생성된 각 값을 연속적으로 계산하지 않음
```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```

## 마치며
* 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
* filter, distinct, takeWhile, dropWhile, skip, limit 메서드로 스트림을 필터링하거나 자를 수 있다.
* 소스가 정렬되어 있다는 사실을 알고 있을 때 takeWhile과 dropWhile 메서드를 효과적으로 사용할 수 있다.
* map, flatMap 메서드로 스트림의 요소를 추출하거나 변환할 수 있다.
* findFirst, findAny 메서드로 스트림의 요소를 검색 할 수 있다. allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색 할 수 있다.
* 이들 메서드는 쇼트셔킷, 즉 결과를 찾는 즉시 반환하며, 전체 스트림을 처리하지 않는다
* reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다. 예를 들어 reduce로 스트림의 최댓값과 모든 요소의 합계를 계산할 수 있다.
* filter, map 등은 상태를 저장하지 않는 사아태 없는 연산이다. reduce 같은 연산은 값을 계산하는 데 필요한 상태를 저장한다. sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야 한다. 이런 메서드를 상태 있는 연산이라고 부른다.
* IntStream, DoubleStream, LongStream은 기본형 특화 스트림이다. 이들 여산은 각각의 기본형에 맞게 특화되어 있다.
* 컬렉션뿐 아니라 값, 배열, 파일, iterate와 generate 같은 메서드로도 스트림을 만들 수 있다.
* 무한한 개수의 요소를 가진 스트림을 무한 스트림이라 한다.