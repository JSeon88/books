# 동작 파라미터화 코드 전달하기

> 동작 파라미터화 : 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록. 코드 블록의 실행은 나중에 실행

## 변화하는 요구사항에 대응하기
### 첫 번째 시도 : 녹색 사과 필터링
```java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (GREEN.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```
    
    * 녹색 말고 빨간 사과를 필터링 하고 싶다면?
        * 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화해야 함.

### 두 번째 시도 : 색을 파마리터화
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }
    }
    return result;
}

List<Apple> greenAples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

* 색 말고 무게를 필터링 하고 싶다면?
    ```java
    public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (apple.getWeight() > weight) {
                result.add(apple);
            }
        }
        return result;
    }
    ```

    * 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복됨.
        * 소프트웨어 공학의 DRY(don't repeat yourself) : 같은 것을 반복하지 말아라 원칙을 어김

### 세 번째 시도 : 가능한 모든 속성으로 필터링
```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}

List<Apple> greenAples = filterApples(inventory, GREEN, 0, true);
List<Apple> redApples = filterApples(inventory, null, 150, false);
```

* 형편 없는 코드. 앞으로 요구사항이 바뀌었을 때 유연하게 대응할 수 없음

## 동작 파라미터화
* 선택 조건을 결정하는 인터페이스를 정의
    ```java
    public inteface ApplePredicate {
        boolean test (Apple apple);
    }

    // 무거운 사과만 선택
    public class AppleHeavyWeightPredicate implements ApplePredicate {
        public boolean test(Apple apple) {
            return apple.getWeight() > 150;
        }
    }

    // 녹색 사과만 선택
    public class AppleGreenColorPredicate implements ApplePredicate {
        public boolean test(Apple apple) {
            return GREEN.equals(apple.getColor());
        } 
    } 
    ```

    * 조건에 따라 filter 메서드가 다르게 동작할 것임.
        * 전략 디자인 패턴
            * ApplePredicate : 알고리즘 패밀리
            * AppleheavyWeightPredicate, AppleGreenColorPredicate가 전략
    > 전략 디자인 패턴 : 각 알고리즘(전략이라 불리는)을 캠슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법.

### 네 번째 시도 : 추상적 조건으로 필터링
```java
// 프레디케이트 객체로 사과 검사 조건을 캡슐화 함.
public static List<Apple> filterApples(List<Apple> inventory, Applepredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

* 전달한 ApplePredicate 객체의 의해 filterApples 메서드의 동작이 결정됨.
* 메서드는 객체만 인수로 받으므로 test 매세더를 ApplePredicate 객체로 감싸서 전달해야 함. test 메서드를 구현하는 객체를 이용해서 불리언 표현식 등을 전달할 수 있음으로 `코드를 전달`할 수 있는 것과 같음.

* 동작 파라미터화의 강점
    * 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리 할 수 있음.

## 복잡한 과정 간소화
### 다섯 번째 시도 : 익명 클래스 사용
> 익명 클래스 : 자바의 지역 클래스(블록 내부에 선언된 클래스)와 비슷한 개념. 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있음.

```java
// filterApples 메서드의 동작을 직접 파마리터화 함.
public static List<Apple> filterApples(List<Apple> inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
});
```

* 단점
    * 익명 클래스는 여전히 많은 공간을 차지함.
        * 장황한 코드는 구현하고 유지보수하는 데 시간이 오래 걸릴 뿐 아니라 쉽게 읽히지 않음.
    * 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않음.

### 여섯 번째 시도 : 람다 표현식 사용
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

### 일곱 번째 시도 : 리스트 형식으로 추상화
```java
public interface Predicate<T> {
    boolean test(T t);
}

// 형식 파라미터 T
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}

List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

## 마치며
* 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행 할 수 있도록 코드를 메서드 인수로 전달한다.
* 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄 일 수 있다.
* 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있따. 하지만 자바 8 이전에는 코드를 지저분하게 구현해야 했다. 익명 클래스로도 어느 정도 코드를 깔끔하게 만들 수 있지만 자바 8에서는 인터페이스를 상속받아 여러 클래스를 구현해야 하는 수고를 없앨 수 있는 방법을 제공한다.
* 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다.