# 2. 람다식

## 2.1 첫 번째 람다식 만들기

예제 2.1 익명의 내부 클래스를 사용하여 버튼 클릭 시 정해진 동작을 수행하는 코드

``` c
butto.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent event) {
    System.out.println("button clicked");
  }
});
```

람다식으로 변경 한 버튼 클릭 시 정해진 동작을 수행하는 코드

``` c
buttom.addActionListener(event -> Systom.out.println("button clicked")
```

* event : 파라미터의 이름
  * 익명 내부 클래스 예제에서의 파라미터와 같음
* -> : 파라미터와 람다식을 분리하는 지시자
* 내부적으로 자바 컴파일러는 코드의 흐름을 통해 event 변수의 타입을 추론해서 해당 타입을 알아냄.

<br>

## 2.2 람다 제대로 활용하기
``` c
Runnable noArguments = () -> System.out.println("Hello World");
```
* 인자가 전혀 없는 람다식
  * 소괄호 () 를 사용하여 인자가 없는 것을 나타냄
  * 인자 없이 실행되고 반환 타입은 void
``` c
ActionListenenr oneArgument = event -> System.out.println("button clicked");
```
* 빈 괄호가 제거되고 하나의 인자를 가지는 람다식
``` c
Runnable multiStatement = () -> {
  System.out.print("Hello");
  System.out.println("World");
};
```
* 간단한 한 줄짜리 람다식을 사용하지 않고 중괄로({})로 둘러싸인 하나의 큰 코드 블록을 사용한 람다식
  * 필요한 값을 반환할 수도 있고, 예외를 블록 밖으로 던질 수도 있음.
``` c
BinaryOperator<Long> add = (x,y) -> x+y;
```
* 하나 이상의 인자를 가진 메소드를 가진 람다식
  * add 는 두 숫자를 더한 결과값을 가지는 BianryOperator<Long> 타입의 변수가 아니라 두 수를 더해서 반환하는 함수
``` c
BinaryOperator<Long> addExplicit = (Long x, Long y) -> x+y;
```
* 람다식의 타입은 컴파일러가 자동 추론 가능하지만 때로는 타입을 직접 명시할 수 있음.

<br>

## 2.3 값의 사용
예제 익명 내부 클래스에 의해 캡쳐된 final 지역 변수
``` c
final String name = getUserName();
butto.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent event) {
    System.out.println("button clicked");
  }
});
```
* 익명 내부 클래스를 사용할 때 클래스 안에 있는 메소드 안에서 메소드 밖의 변수를 사용해야 하는 경우 변수를 **final** 로 만들어야 함
``` c
String name = getUserName();
buttom.addActionListener(event -> Systom.out.println("hi " + name));
```
* 변수를 final로 선언하지는 않았지만 해당 변수가 람다식 안에서 사용되고 있다면 그 변수는 내부적으로 final 변수로 바뀜
* **람다식이 변수를 캡쳐하는 것이 아니라 값을 캡쳐하기 때문**

<br>

## 2.4 함수형 인터페이스

> **람다식의 타입으로 사용되는 하나의 추상 메소드를 가진 인터페이스**

예제 ActionEvent를 받고 반환값이 없는 ActionListener 인터페이스 코드
``` c
public interface ActionListener extends EventListener {
  public void actionPerformed(ActionEvent event);
}
```
자바에서 중요한 함수형 인터페이스   

| 인터페이스 이름       | 인자   | 반환     |
| ----------------- | ----- | ------- |
| Predicate<T>      | T     | boolean |
| Consumer<T>       | T     | void    |
| Funtion<T,R>      | T     | R       |
| Supplier<T>       | None  | T       |
| UnaryOperator<T>  | T     | T       |
| BinaryOperator<T> | (T,T) | T       |

<br>

## 2.5 타입 인터페이스
> 타입을 직접 제공해야 하는지 아닌지를 언제 알 수 있을까?
예제 2.9 변수의 타입 추측을 위한 다이아몬드 연산자 사용 코드
``` c
Map<String, Integer> oldWordCounts = new HashMap<String, Integer>();
Map<String, Integer> diamondWordCounts = new HashMap<>();
```
* 생성자를 메소드 호출 구분에서 직접 사용한다면 컴파일러는 메소드에 선언된 일반적 타입으로 생성자의 타입을 추론
예제 2.14 일반적 타입이 명시되지 않아서 컴파일되지 않는 코드
``` c
BinaryOperator add = (x,y) -> x+y;
```
```
Operator '&#x002B;' cannot be applied to java.lang.Object, java.lang,Object."
```
* BinaryOperator는 일반적 타입의 인자를 가진 함수형 인터페이스
* x,y의 두개의 인자는 반환값의 타입으로도 사용
  * 즉, 사용할 변수에 어떠한 일반적 타입도 명시해 주지 않았는데 이렇게 되면 자바에서 가장 기본이 되는 타입으로 추론   
컴파일러는 인자들과 반환값이 java.lang.Object의 인스턴스라고 잘못 생각하게 됨.

<br>

## 정리
- 람다식은 데이터처럼 넘기기 위해 사용되는 이름 없는 메소드
- 람다식 형식
```
BinaryOperator<Long> add = (x,y) -> x+y;
```
- 함수형 인터페이스는 람다식의 타입처럼 사용되는 하나의 추상 메소드를 가진 인터페이스
