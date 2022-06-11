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

## 함수형 인터페이스 사용
> 함수형 인터페이스는 오직 하나의 추상 메서드를 지정

> 함수 디스크립터 : 함수형 인터페이스의 추상 메서드 시그니처

### Predicate
* java.util.function.Predicate<T> 인터페이스
* `test` 라는 추상 메서드를 정의
* 제네릭 형식 T의 객체를 인수로 받아 `불리언`을 반환
```java
@FunctionInterface
public interface Predicate<T> {
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for (T t : list) {
        if (p.test(t)) {
            results.add(t);
        }
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

### Consumer
* java.util.function.Consumer<T> 인터페이스
* 제네릭 형식 T 객체를 받아서 `void`를 반환하는 `accept`라는 추상 메서드를 정의
* T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스를 사용할 수 있음
```java
@FunctionInterface
public interface Consumer<T> {
    void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accept(t);
    }
}

forEach(Arrays.asList(1,2,3,4,5), (Integer i) -> System.out.println(i));
```

### Function
* java.util.function.Function<T,R> 인터페이스
* 제네릭 형식 T를 인수로 받아서 `제네릭 형식 R 객체를 반환`하는 추상 메서드 `apply`를 정의
* 입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있음
```java
@FunctionInterface
public interface Function<T,R> {
    R apply(T t);
}

public <T,R> List<T> map(List<T> list, Function<T,R> f) {
    List<R> result = new ArrayList<>();
    for (T t : list) {
        result.add(f.apply(t));
    }
    return result;
}

// [7,2,6]
List<Integer> l = map(Arrays.asList("lambdas","in","action"), (String s) -> s.length());
```

#### 기본형 특화
* 제네릭 특성 상 제네릭 파라미터에는 참조형만 사용할 수 있음
* 자바에서는 기존형을 참조형으로 변환하는 기능을 제공(박싱). 그 반대인 언박싱 또한 제공
    * 박싱한 값은 기본형을 감싸는 래퍼며 힙에 저장. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요
* 오토박싱 동작을 피할 수 있또록 특별한 버전의 함수형 인터페이스를 제공

|함수형 인터페이스|함수 디스크립터|기본형 특화|
|------|---------|------|
|Predicate<T>|T -> boolean|`IntPredicate, LongPredicate, DoublePredicate`|
|Consumer<T>|T -> void|`IntConsumer, LongConsumer, DoubleConsumer`|
|Function<T,R>|T -> R|`IntFunction<R>, IntToDoubleFunction, IntToLongFunction, LongFunction<R>, LongToDoubleFunction, LongToIntFunction, DoubleFunction<R>, DoubleToIntFunction, DoubleToLongFunction, ToIntFunction<R>, ToDoubleFunction<R>, ToLongFunction<R>`|
|Supplier<T>|() -> T|`BooleanSupplier, IntSupplier, LongSupplier, DoubleSupplier`|
|UnaryOperator<T>|T -> T|`IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator`|
|BinaryOperator<T>|(T, T) -> T|`IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator`|
|BiPredicate<L,R>|(T, U) -> boolean||
|BiConsumer<T,U>|(T, U) -> void|`ObjIntConsumer<T>, ObjLongConsumer<T>, ObjDoubleConsumer<T>`|
|BiFunction<T,U,R>|(T, U) -> R|`ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>`|

#### 예외, 람다, 함수형 인터페이스와의 관계
* 예외를 던지는 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try/catch 블록으로 감싸야 함
```java
// 예외를 던지는 람다 표현식
@FuntionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();

// 람다를 tyr/catch 블록으로 감싸기
Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    } catch(IOException e) {
        throw new RuntimeException(e);
    }
};
```

## 형식 검사, 형식 추론, 제약
### 형식 검사
```java
filter(inventory, (Apple a) -> a.getWeigth() > 150);

// 1. 람다가 사용된 콘텍스트는 무엇인가? 우선 filter의 정의를 확인

filter(List<Apple> inventory, Predicate<Apple> p)

// 2. 대상 형식은 Predicate<Apple>. T는 Apple로 대치

// 3. Predicate<Apple> 인터페이스의 추상 메서드는 무엇인가?

boolean test(Apple apple)

// 4. Apple을 인수로 받아 boolean을 반환하는 test 메서드

Apple -> boolean

// 5. 함수 디스크립터는 Apple -> boolean이므로 람다의 시그니처와 일치. 람다도 Apple을 인수로 받아 boolean을 반환하므로 코드 형식 검사가 성공적으로 완료
```

### 같은 람다, 다른 함수형 인터페이스
* 하나의 람다 표현식을 다양한 함수형 인터페이스에 사용할 수 있음
```java
Comparator<Apple> c1 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
ToIntBiFunction<Apple, Apple> c2 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
BiFunction<Apple, Apple, Integer> c3 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

* 특별한 void 호환 규칙
    * 람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환(파라미터 리스트도 호환되어야 함)
    ```java
    // Predicate는 불리언 반환값을 가짐
    Predicate<String> p = s -> list.add(s);
    // Consumer는 void 반환값을 가짐
    Consumer<String> b = s -> list.add(s); // void 대신 boolean을 반환하지만 유효한 코드
    ```

### 형식 추론
* 람다 표현식이 사용된 컨텍스트(대상 형식)를 이용해서 람다 표현식과 함수형 인터페이스를 추론
```java
// 형식을 추론하지 않음
Comparator<Apple> c1 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
// 형식을 추론함
Comparator<Apple> c1 = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

### 지역 변수 사용
* 람다 표현식에서는 익명 함수가 하는 것처럼 `자유 변수`(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있음
* 지역 변수의 제약 조건
    * 명시적으로 `final`로 선언되거나 final로 선언된 변수와 똑같이 사용되어야 함
* 왜 제약조건이 생긴 걸까?
    * 인스턴스 변수 : 힙에 저장
    * 지역 변수 : 스택에 위치
    * 람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제 되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근할려는 문제가 생김
        * 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공

## 메서드 참조
```java
// 기존 코드
inventory.sort((Apple a1, Apple a2) -> a1.getWight().compareTo(a2.getWeigth()));
// 메서드 참조. java.util.Comparator.comparing을 활용
inventory.sort(comparing(Apple::getWeight));
```

#### 메서드 참조는 왜 중요한가?
* 람다가 이 메서드를 직접 호출해라고 명령한다면 메서드를 어떻게 호출해야 하는지 설명을 참조하기보다는 메서드명을 직접 참조하는 것이 편리
    * 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있음

### 메서드 참조를 만드는 방법
* 정적 메서드 참조
    * Integer의 parseInt 메서드는 Integer::parseInt
* 다양한 형식의 인스턴스 메서드 참조
    * String의 length 메서드는 String::length
* 기존 객체의 인스턴스 메서드 참조
    * Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고, Trasaction 객체에는 getValue 메서드가 있다면, expensiveTransaction::getValue

### 생성자 참조
```java
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();    // Supplier의 get 메서드를 호출해서 Apple 객체를 생성

// 위의 코드는 다음과 같음
Supplier<Apple> c1 = () -> new Apple();     // 람다 표현식은 디폴트 생성자를 가진 Apple을 만듬
Apple a1 = c1.get();
```
```java
Function<Integer, Apple> c2 = Apple::new;   // Apple(Integer, weight)의 생성자 참조
Apple a2 = c2.apple(110);   // Function의 apply 메서드에 무게를 인수로 호출해서 새로운 Apple 객체를 생성

// 위의 코드는 다음과 같음
Function<Integer, Apple> c2 = (weight) -> new Apple(weight);
Apple a2 = c2.apple(110); 
```

* 인스턴스화하지 않고도 생성자에 접근할 수 있는 기능을 다양한 상황에 응용할 수 있음
    * Map으로 생성자와 문자열 값을 관련 시키는 예제
    ```java
    static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
    static {
        map.put("apple", Apple::new);
        map.put("orange", Orange::new);
        // 등등
    }

    public static Fruit giveMeFruit(String fruit, Integer weight) {
        return map.get(fruit.toLowerCase())     // map에서 Function<Integer, Fruit>를 얻음
                  .apply(weight);    // Function의 apply 메서드에 정수 무게 파라미터를 제공해서 Fruit를 생성
    }
    ```

## 람다, 메서드 참조 활용하기
* 앞선 예제의 사과 리스트를 다양한 정렬 기법으로 정렬하는 문제의 코드를 수정

### 1단계 : 코드 전달
```java
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}

inventory.sort(new AppleComparator());
```

### 2단계 : 익명 클래스 사용
```java
inventory.sort(new Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```

### 3단계 : 람다 표현식 사용
* 람다 표현식이라는 경량화된 문법을 이용해서 코드를 전달 할 수 있음
* (Apple, Apple) -> int 로 표현 가능
    ```java
    inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

    // 형식 추론으로 변경
    inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
    ```
* Comparator는 Comparable 키를 추출해서 Comparator 객체로 만드는 Function 함수를 인수로 받는 정적 메소드 comparing을 포함함. comparing 메서드를 사용
    ```java
    Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight())

    // 간소화하게 되면
    import static java.util.Comparator.comparing;
    inventory.sort(comparing(apple -> apple.getWeight()));
    ```
### 4단계 : 메서드 참조 사용
```java
import static java.util.Comparator.comparing;
inventory.sort(comparing(Apple::getWeight);
```

* 코드 자체로 Apple을 weight 별로 비교해서 inventory를 sort하라는 의미를 전달 할 수 있음

## 람다 표현식을 조합할 수 있는 유용한 메소드
* Comparator, Function, Predicate 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있도록 유틸리티 메서드를 제공. 즉, 간단한 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있음
    * 디폴트 메서드가 있기 때문에 가능

### Comparator 조합
#### 역정렬
```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

#### Comperator 연결
* 무게가 같을 경우 원산지 국가별로 정렬은 어떻게 할 수 있을까?
    * 두 번째 비교자를 만들 수 있음(thenComparing)
    ```java
    inventory.sort(comparing(Apple::getWeight)
             .reversed()
             .thenComparing(Apple::getCountry));
    ```

### Predicate 조합
* 복잡한 프레디케이트를 만들 수 있도록 세가지 메서드 제공
    * negate
    ```java
    // 기존 프레디케이트 객체 redApple의 결과를 반전시킨 객체 생성
    Predicate<Apple> notRedApple = redApple.negate();
    ```
    * and
    ```java
    // 두 프로디케이트를 연결해서 새로운 프레디케이트 객체를 생성
    Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);
    ```

    * or
    ```java
    // 프레디케이트 메서드를 연결해서 더 복잡한 프레디케이트를 생성
    Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(apple -> apple.getWeight() > 150)
                                                       .or(apple -> GREEN.equals(a.getColor()));
    ```

### Function 조합
* andThen
    * 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 합수
    ```java
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = x -> f.andThen(g);   //g(f(x))

    int result = h.apply(1);    // 4 반환
    ```
* compose
    * 인수로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공
    ```java
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = x -> f.compose(g);   //f(g(x))

    int result = h.apply(1);    // 4 반환
    ```

## 마치며
* 람다 표현식은 익명 함수의 일종. 이름은 없지만, 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.
* 람다 표현식으로 간결한 코드를 구현 할 수 있다.
* 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스이다.
* 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며, 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
* java.util.function 패키지는 Predicate<T>, Function<T,R>, Supplier<T>, Consumer<T>, BinaryOperator<T> 등을 포함해서 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
* 자바 8은 Predicate<T>와 Function<T,R> 같은 제네릭 함수형 인터페이스와 관련된 박싱 동작을 피할 수 있는 IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공한다.
* 실행 어라운드 패턴(예를 들어 자원 할당, 자원 정리 등 코드 중간에 실행해야 하는 메서드에 꼭 필요한 코드)을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
* 람다 표현식의 기대형식을 대상 형식이라고 한다.
* 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달 할 수 있다.
* Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.