# 스트림 소개

## 스트림이란 무엇인가?
* 자바 8 API에 새로 추가된 기능
* 선언형(즉, 데이터를 처리하는 임시 구현 코드 대신 질의로 표현)으로 컬렉션 데이터를 처리 가능
* 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리 가능

* 기존 코드
    ```java
    // 가비지 변수 사용(lowCaloricDishes)
    List<Dish> lowCaloricDishes = new ArrayList<>();
    for (Dish dish : menu) {
        if (dish.getCalories() < 400) {
            lowCaloricDishes.add(dish);
        }
    }

    Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
        public int compare(Dish dish1, Dish dish2) {
            return Integer.compare(dish1.getCalories(), dish2.getCalories());
        }
    });

    List<String> lowCaloricDishesName = new ArrayList<>();
    for (Dish dish : lowCaloricDishes) {
        lowCaloricDishesName.add(dish.getName());
    }
    ```
* 바뀐 코드
    ```java
    import static java.util.Comparator.comparing;
    import static java.util.stream.Collectors.toList;

    List<String> lowCaloricDishes = menu.stream()
                                        .filter(d -> d.getCalories() < 400)
                                        .sorted(comparing(Dish::getCalories))
                                        .map(Dish::getName)
                                        .collect(toList());
    

    // stream()을 parallelStream()으로 바꾸면 멀티코어 아키텍처에서 병렬로 실행 함
    List<String> lowCaloricDishes = menu.parallelStream()
                                        .filter(d -> d.getCalories() < 400)
                                        .sorted(comparing(Dish::getCalories))
                                        .map(Dish::getName)
                                        .collect(toList());
    ```
* 스트림의 새로운 기능이 소프트웨어공학적으로 주는 이득은?
    * 선언형으로 코드를 구현 가능
        * 루프와 if 조건문 등의 제어 블록을 사용해서 어떤 동작을 구현할지 지정할 필요 없음.
        * 즉, 선언형 코드와 동작 파라미터화를 활용하면 변하는 요구사항에 쉽게 대응 가능
    * filter, sorted, map, colloct 같은 여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인 생성 가능
    * 스트림 API로 인해 데이터 처리 과정을 병렬화하면서 스레드와 락을 걱정 할 필요가 없음

## 스트림 시작하기
> 스트림 : 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

* 연속된 요소
    * 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공
    * 컬렉션의 주제는 데이터고, 스트림의 주제는 계산
* 소스
    * 스트리은 컬렉션, 배열, I/O 자원등의 데이터 제공 소스로부터 데이터를 소비
    * 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지
* 데이터 처리 연산
    * 한수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스의 비슷한 연산을 지원
    * filter, map, reduce, finde, match, fort 등등
    * 연산은 순차적으로 또는 병렬로 실행 가능

### 스트림의 특징
* 파이프라이닝(Pipelining)
    * 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환
* 내부 반복
    * 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원

### 스트림 연산
* filter
    * 람다를 인수로 받아 스트림에서 특정 요소를 제외
* map
    * 람다를 이용해서 한 요소를 다른 요소로 반환하거나 정보를 추출
* limit
    * 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소 truncate 함
* collect
    * 스트림을 다른 형식으로 변환

## 스트림과 컬렉션
* 데이터를 언제 계산하냐가 가장 큰 차이
    * 컬렉션
        * 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료 구조
    * 스트림
        * 이론적으로 요청할 때만 요소를 계산하는 고정된 자료 구조
            * 스트림에 요소를 추가하거나 스트림에서 요소를 제거할 수 없음
        
### 딱 한 번만 탐색할 수 있다
* 스트림도 한 번만 탐색 할 수 있음
    * 탐색된 스트림의 요소를 소비
        ```java
        List<String> title = Arrays.asList("java8","In","Action");

        Stream<String> s = title.stream();
        s.forEach(System.out::println);     // title의 각 단어를 출력
        s.forEach(System.out::println);     // java.lang.illegalStateException(스트림이 이미 소비되었거나 닫힘) 발생
        ```

### 외부 반복과 내부 반복
* 컬렉션 : for-each 루프를 이용하는 외부 반복
    ```java
    List<String> names = new ArrayList<>();

    for (Dish dish : menu) {    // 메뉴 리스트를 명시적으로 순차 반복
        names.add(dish.getName());
    }
    ```
* 스트림 : 내부 반복
    ```java
    List<String> names = menu.stream()
                             .map(Dish::getName)
                             .collect(toList());
    ```
## 스트림 연산
* 중간 연산 : 연결 할 수 있는 스트림 연산
* 최종 연산 : 스트림을 닫는 연산

### 중간 연산
* filter나 sored 같은 중간 연산은 다른 스트림을 반환
* 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않음(Lazy)
```java
 List<String> lowCaloricDishes = menu.stream()
                                     .filter(dish -> {
                                        System.out.println("filtering:" + dish.getName())
                                        return dish.getCalories() > 300
                                     })
                                     .map(dish -> {
                                        System.out.println("mapping:" + dish.getName())
                                        return dish.getName();
                                     })
                                     .list(3)
                                     .collect(toList());
System.out.println(names);


// result
filtering:pork
mapping:prok
filtering:beef
mapping:beef
filtering:chicken
mapping:chicken
[prot, beef, checken]
```

* 300 칼로리가 넘는 요리가 여러개지만 게으른 특성으로 인해 오직 처음 3개만 선택
    * `쇼트서킷` 기법 덕분
* filter, map은 서로 다른 연산이지만 한 과정으로 병합 됨
    * `루프 퓨전`

### 최종 연산
* 스트림 파이프라인에서 결과를 도출

### 스트림 이용하기
* 스트림 이용 과정
    * 질의를 수행할 (컬렉션 같은) 데이터 소스
    * 스트림 파이프라인을 구성할 중간 연산 연결
    * 스트림 파이프라인을 실행하고 결과를 만들 최종 연산

## 마치며
* 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
* 스트림은 내부 반복을 지원한다. 내부 반복은 filter, map, sorted 등의 연산으로 반복을 추상화한다.
* 스트림에는 중간 연산과 최종 연산이 있다
* 중간 연산은 filter와 map처럼 스트림을 반환하면서 다른 연산과 연결되는 연산이다. 중간 연산을 이용해서 파이프라인을 구성할 수 있지만 중간 연산으로는어떤 결과도 생성 할 수 없다.
* forEach나 count처럼 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산을 최종 연산이라고 한다.
* 스트림의 요소는 요청할 때 게으르게 계산된다.