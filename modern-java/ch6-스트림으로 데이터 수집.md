# 스트림으로 데이터 수집
## 컬렉터란 무엇인가?
* Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정
* 필요한 컬렉터를 쉽게 추가 가능

### 고급 리듀싱 기능을 수행하는 컬렉터
* 강점 : 높은 수준의 조합성과 재사용성
* 스트림에 collect를 호출하면 스트림의 요소에 리듀싱 연산 수행
* Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 메서드 제공

### 미리 정의된 컬렉터
* Collectors에서 제공하는 메서드의 기능
    * 스트림 요소를 하나의 값으로 리듀스하고 요약
        * 트랜잭션 리스트에서 트랜잭션 총합을 찾는 등의 다양한 계산을 수행할 때 유용함
    * 요소 그룹화
    * 요소 분할
        * 한 개의 인수를 받아 불리언을 반환하는 함수, 즉 프레디케이트를 그룹화 함수로 사용

## 리듀싱과 요약
* 컬렉터(Stream.collect 메서드의 인수)로 스트림의 항목을 컬렉션으로 재구성 가능
* 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있음
* 얘제코드 : counting()
    ```java
    long howManyDishes = menu.stream().collect(Collectors.counting());

    // 불필요한 과정 생략
    long howManyDishes = menu.stream().count();
    ```

### 스트림 값에서 최댓값과 최솟값 검색
* Collectors.mayBy : 최댓값
* Collectors.minBy : 최솟값
```java
// Collectors.mayBy 예제

Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream()
                                     .collect(maxBy(dishCaloriesComparator));
```

### 요약 연산
> 요약(summarization) 연산 : 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산
* Collectors.summingInt
    ```java
    int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
    ```
    * 칼로리로 매핑된 각 요리의 값을 탐색하면서 초기값(여기서는0)으로 설정되어 있는 누적자에 칼로리를 더함
* Collectors.summingLong, Collectors.sumingDouble 메서드도 같은 방식으로 동작
* 두 개 이상의 연산을 한번에 수행해야 할 때 : summarizingInt
    ```java
    IntSummaryStatistics menuStatistics = menu.stream()
                                              .collect(summarizingInt(Dish::getCalories));

    // result
    IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}
    ```
### 문자열 연결
* joining
    * 스트림의 각 객체에 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환
    ```java
    String shrotMenu = menu.stream()
                           .map(Dish::getName)
                           .collect(joining());
    ```
    * joining 메서드는 내부적으로 StringBuilder를 이요해서 문자열을 하나로 생성
    * Dish 클래스가 요리명을 반환하는 toString 메서드를 포함하고 있다면 map 생략 가능
        ```java
        String shortMenu = menu.stream()
                               .collect(joining());

        // result
        prokbeefcheckenfrench friesriceseason fruitpizzaprawnssalmon
        ```
    * 구분 문자열 추가
        ```java
        String shortMenu = menu.stream()
                               .map(Dish::getName)
                               .collect(joining(", "));

        // result
        prot, beef, chickent, french fries, season fruit, pizza, prawns, salmon
        ```

### 범용 리듀싱 요약 연산
* 소개한 코든 컬렉터는 reducing 팩터리 메서드로도 정의 할 수 있음
    * Collectors.reducing
        * 메뉴의 모든 칼로리 합계 계산 예제
            ```java
            int totalCalories = mewnu.stream()
                                    .collect(reducing(0, Dish::getCalories, (i,  j) -> i + j));
            ```
            * reducing 인수
                * 첫 번째 인수 : 리듀싱 연산의 시작값이거나 스트림에 인수가 없을때는 반환값
                * 두 번째 인수 : 변환 함수
                    * 예제에서는 요리를 칼로리 정수로 변환
                * 세 번째 인수 : 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator
        * 칼로리가 높은 요리를 찾는 예제
            ```java
            Optional<Dish> mostCalorieDish = menu.stream()
                                                 .collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2))
            ```
            * 세 개의 인수를 갖는 reducing 메서드에서 스트림의 첫 번째 요소를 시작 요소, 즉 첫 번째 인수로 받으며, 자신을 그대로 반환하는 `항등 함수`를 두 번째 인수로 받는 상황에 해당

#### collect와 reduce
* 의미론적인 문제
    * collect
        * 도출하려는 결과를 누적하는 컨테이너를 변경하도록 설계
    * reduce
        * 두 값을 하나로 도출하는 불변형 연산
* 실용적인 문제
    * 리듀싱 연산을 병렬로 수행 불가능

> 병렬성을 확보하려면 collect 메서드로 리듀싱연산을 구현하는 것이 바람직

#### 컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행 할 수 있다.
* Integer 클래스의 sum 메서드
    ```java
    int totalCalories = menu.stream()
                            .collect(reducing(0, Dish::getCalories, Integer::sum));     // 초기값, 합계 함수, 변환 함수
    ```
* 요리 스트림을 요리의 칼로리로 매핑한 다음에 메서드 참조로 결과 스트림을 리듀싱
    ```java
    int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get
    ```
    * 기본값을 제공할 수 있는 orElse, orElseGet 등을 이용해서 Oprional의 값을 얻어오는 것이 좋음
* 스트림을 IntStream으로 매핑한 다음에 sum 메서드를 호출하는 방법
    ```java
    int totalCalories = menu.stream()
                            .mapToInt(Dish::getCalories).sum();
    ```

#### 자신의 상황에 맞는 최적의 해법 선택
* 가독성과 성능을 잡기 위해서는 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직
* 현재 예제에서는 마지막 방법(IntStream을 이용)이 가독성이 가장 좋고 간결함. 또한 자동 언박싱 연산을 수행하거나 Integer를 int로 반환하는 과정을 피할 수 있어 성능까지 좋음.

## 그룹화
* Collectors.groupingBy
    ```java
    Map<Dish, Type, List<Dish>> dishesByType = menu.stream()
                                                   .collect(groupingBy(Dish::getType));

    // result
    {FISH=[parawns, salmon], OTHER={french fries, rice, season fruit, pizza], MEAT=[port, beef, chicken]}
    ```
    * 분류 함수 : 함수를 기준으로 스트림이 그룹화
    * 400 칼로리를 'diet', 400~700칼로리를 'normal', 700칼로리 초과를 'fat'요리로 구분하는 예제
        ```java
        public enum CaloricLevel { DIET, NORMAL, FAT }

        Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(groupingBy(dish -> {
            if (dish.getCalories() <=400) return CaloricLevel.DIET;
            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
            else return CaloricLevel.FAT;
        }));
        ```

### 그룹화된 요소 조작
* 500 칼로리가 넘는 요리만 필터링하는 예제
    ```java
    Map<Dish, Type, List<Dish>> caloricDishesByType = menu.stream()
                                                          .filter(dish -> dish.getCalories() > 500)
                                                          .collect(groupingBy(Dish::getType));

    // result
    {OTHER=[french fries, pizza], MEAT=[pork, beef]}
    ```
    * filter을 사용했을 경우 단점
        * 필터 조건에 만족하는 FISH 종류 요리가 없으므로 결과 맵에서 해당 키 자체가 사라짐
        * 해결 방법
            * collector 안으로 필터 프레디케이트를 이동
                ```java
                Map<Dish, Type, List<Dish>> caloricDishesByType = menu.stream()
                                                                      .collect(groupingBy(Dish::getType, 
                                                                                          filtering(dish -> dish.getCalories() > 500, toList())));

                // result
                {OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}
                ```
* 맵의 각 그룹이 문자열 리스트라면?
    ```java
    Map<String List<String>> dishTags = new HashMap<>();
    dishTags.put("pork", asList("greasy","salty"));
    dishTags.put("beef", asList("salty", "roasted"));
    dishTags.put("chicken", asList("fried","crisp"));
    dishTags.put("french fries", asList("greasy","fried"));
    dishTags.put("rice", asList("light","natural"));
    dishTags.put("season fruit", asList("fresh","natural"));
    dishTags.put("pizza", asList("tasty","salty"));
    dishTags.put("prawns", asList("tasty","roasted"));
    dishTags.put("salmon", asList("delicious","fresh"));
    ```
    * 각 형식의 요리의 태그 추출 예제
        ```java
        Map<Dish.Type, Set<String>> dishNamesByType = menu.stream()
                                                          .collect(groupingBy(Dish::getType, 
                                                                              flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));
        // result
        {MEAT=[salty, greasy, roasted, fried, crisp]. FISH=[roasted, tasty, fresh, delicious], OTHER=[salty, greasy, natural, light, tasty, fresh, freid]}
        ```
        * flatMapping 연산 결과를 수집해서 리스트가 아니라 집합으로 그룹화해 중복 태그를 제거

### 다수준 그룹화
* Collectors.groupingBy 인수
    * 분류 함수
    * 컬렉터
* 바깥쪽 groupingBy 메서드에 스트림의 항목을 분류할 두 번째 기준을 정의하는 내부 groupingBy를 전달해서 두 수준으로 스트림의 항목을 그룹화 하는 예제
    ```java
    Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishsByTypeCaloricLevel = 
    menu.stream()
        .collect(groupingBy(Dish::getType, 
                            groupingBy(dish -> {
                                            if (dish.getCalories() <= 400) {
                                                return CaloricLevel.DIET;
                                            } else if (dish.getCalories() <= 700) {
                                                return CaloricLevel.NORMAL;
                                            } else {
                                                return CaloricLevel.FAT;
                                            }
                            })));
    
    // result
    {MEAT={DIET=[chicken], NOMAL=[beef], FAT=[pork]}, 
    FISH={DIET=[prawns], NOMAL=[salmon]}, 
    OTHER={DIET=[rice, seasonal fruit], NOMAL=[french fries, pizza]}}
    ```

### 서브그룹으로 데이터 수집
* 첫 번째 groupingBy로 넘겨주는 컬렉터의 형식은 제한 없음
* 메뉴에서 요리의 수를 종류별로 계산하는 예제
    ```java
    Map<Dish.Type, Long> typesCount = menu.stream()
                                          .collect(groupingBy(Dish::getType, counting()));

    // result
    {MEAT=3, FISH=3, OTHER=4}
    ```
    * groupingBy 컬렉터에 두 번째 인수로 counting 컬렉션을 전달.
    * 앞선 예제에서 groupingBy(f)는 사실 groupingBy(f, toList())의 축약형
* 요리의 종류를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 예제
    ```java
    Map<Dish.Type, Optional<Dish>> mostCaloricByType = menu.stream()
                                                           .collect(groupingBy(Dish::getType, 
                                                                                maxBy(comparingInt(Dish::getCalories))));

    // result
    {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
    ```
    * maxBy가 생성하는 컬렉터의 결과 형식에 따라 맵의 값이 Optional 형식이 됨.
        * 메뉴의 요리에 Optional.empty()인 요리가 존재하지 않으므로 Optional 래퍼를 사용할 필요가 없음

#### 컬렉터 결과를 다른 형식에 적용하기
* Collectors.collectingAndThen
    ```java
    Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
                                                 .collect(groupingBy(Dish::getType, 
                                                            collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));

    // result
    {FISH=salmon, OTHER=pizza, MEAT=pork}  
    ```
    * 팩토리 메서드 collectingAndThen는 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환

#### groupingBy와 함께 사용하는 다른 컬렉터 예제
* 스트림에서 같은 그룹으로 분류된 모든 요소에 리듀싱 작업을 수행할 때는 팩토리 메서드 groupingBy에 두 번째 인수로 전달한 컬렉터를 사용
* 메뉴에 있는 모든 요리의 칼로리 합계를 구하려고 만든 컬렉터를 재사용한 예제
    ```java
    Map<Dish.Type, Integer> totalCaloriesByType = menu.stream()
                                                      .collect(groupBy(Dish::getType, summingInt(Dish::getCalories)));
    ```
* 각 요리 형식에 존재하는 모든 CaloricLevel 값 구하는 예제
    ```java
    Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = menu.stream()
                                                                .collect(groupingBy(Dish::getType, 
                                                                                    mapping(dish -> {
                                                                                        if (dish.getCalories() <= 400) {
                                                                                            return CaloricLevel.DIET;
                                                                                        } else if (dish.getCalories() <= 700) {
                                                                                            return CaloricLevel.NORMAL;
                                                                                        } else {
                                                                                            return CaloricLevel.FAT;
                                                                                        }
                                                                                    }, toSet())));
    // result
    {OTHER=[DIET, NORMAL], MEAT=[DIET, NORMAL, FAT], FISH=[DIET, NORMAL]}
    ```
    * mapping은 입력 요소를 누적하기 전에 매핑 함수를 적용해서 다양한 형식의 객체를 주어진 형식의 컬렉터에 맞게 변환하는 역할을 함
    * mapping 메서드에 전달한 변환 함수는 Dish를 CaloricLevel로 매핑. CaloricLevel 결과 스트림은 toSet 컬렉터로 전달되면서 리스트가 아닌 집합으로 스트림의 요소가 누적됨.

## 분할
> 분할 : `분할 함수`라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능. 반환은 불리언
```java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetarian));

// result
{false=[pork, beef, chicken, prawns, salmon], 
true=[french fries, rice, season fruit, pizza]}

List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

### 분할의 장점
* 분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지
    * 컬렉터를 두 번째 인수로 전달할 수 있는 오버로드된 버전의 partitoningBy 메서드 얘제
        ```java
        Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream()
                                                                            .collect(partitioningBy(Dish::isVegetarian, 
                                                                                                    groupingBy(Dish::getType)));
        
        // result
        {false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},
        true={OTHER=[french fries, rice, season fruit, pizza]}
        ```
    * 이전 코드를 활용하면 채식 요리와 채식이 아닌 요리 각각의 그룹에서 가장 칼로리가 높은 요리를 찾는 것도 가능
        ```java
        Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream()
                                                                            .collect(partitioningBy(Dish::isVegetarian, 
                                                                                                    collectingAndThen(maxBy(comparingInt(Dish::getCalories)),
                                                                                                                                        Optional::get)));
        
        // result
        {false=pork, true=pizza}
        ```
* 내부적으로 partitioningBy는 특수한 맵과 두 개의 필드로 구현
* 다수준으로 분할 가능

#### Collectors 클래스의 정적 팩토리 메서드
|팩토리 메서드|반환 형식|사용 예제|활용 예|
|-----|-----|----|----|
|toList|List<T>|스트림의 모든 항목을 리스트로 수집|`List<Dish> dishes = menuStream.collect(toList());`|
|toSet|Set<T>|스트림의 모든 항목을 중복이 없는 집합으로 수집|`Set<Dish> dishes = menuStream.collect(toSet());`|
|toCollection|Collection<T>|스트림의 모든 항목을 발행자가 제공하는 컬렉션으로 수집|`Collection<Dish> dishes = munuStream.collect(toCollection(), ArrayList::new);`|
|counting|Long|스트림의 항목 수 계산|`long howManyDishes = menuStream.collect(counting());`|
|summingInt|Integer|스트림의 항목에서 정수 프로퍼티값을 더함|`int totalCalories = menuStream.collect(summingInt(Dish::getCalories));`|
|averagingInt|Double|스트림 항목의 정수 프로퍼티의 평균값 계산|`double avgCalories = menuStream.collect(averagingInt(Dish::getCalories));`|
|summarizingInt|Double|스트림 내 항목의 최댓값, 최솟값, 합계, 평균 등의 정수 정보 통계 수집|`IntSummaryStatistics menuStatistics = menuStream.collect(summarizingInt(Dish::getCalories));`|
|joining|String|스트리의 각 항목에 toString 메서드를 호출한 결과 문자열 연결|`String shortMenu = menuStream.map(Dish::getName).collect(joining(", "));`|
|maxBy|Optional<T>|주어진 비교자를 이용해서 스트림의 최댓값 요소를 Optional로 감싼 값을 반환. 스트림에 요소가 없을 때는 Optional.empty() 반환|`Optional<Dish> fattest = menuStream.collect(maxBy(comparingInt(Dish::getCalories)));`|
|minBy|Optional<T>|주어진 비교자를 이용해서 스트림의 최솟값 요소를 Optional로 감싼 값을 반환. 스트림에 요소가 없을 때는 Optional.empty() 반환|`Optional<Dish> fattest = menuStream.collect(minBy(comparingInt(Dish::getCalories)));`|
|reducing|The type produced by the reduction operation|누적자를 초깃값으로 설정한 다음에 BinaryOperator로 스트림의 각 요소를 반복적으로 누적자와 합쳐 스트림을 하나의 값으로 리듀싱|`int totalCalories = menuStream.collect(reducing(0, Dish::getCalories, Integer::sum));`|
|collectingAndThen|The type produced by the reduction operation|다른 컬렉터를 감싸고 그 결과에 변환 함수 적용|`int howManyDishes = menuStream.collect(collectingAndThen(toList(), List::size));`|
|groupingBy|Map<K, List<T>>|하나의 프로퍼티값을 기준으로 스트림의 항목을 그룹화하며 기준 프로퍼티값을 결과 맵의 키로 사용|`Map<Dish.Type, List<Dish>> dishesByType = menuStream.collect(groupingBy(Dish::getType));`|
|partitioningBy|Map<Boolean, List<T>|프레디케이트를 스트림의 각 항목에 적용한 결과로 항목 분할|`Map<Boolean, List<Dish>> vegetarianDishes = menuStream.collect(partitioningBy(Dish::isVegetarian));`|

## Collector 인터페이스
```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```
* T는 수직된 스트림 항목의 제네릭 형식
* A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
* R은 수집 연산 결과 객체의 형식

### Collector 인터페이스의 메서드 살펴보기
* 네 개의 메서드는 collect 메서드에서 실행하는 함수를 반환
* 다섯 번째 메서드 characteristics는 collect 메서드가 어떤 최적화를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공

#### supplier 메서드 : 새로운 결과 컨테이너 만들기
* 빈 결과로 이루어진 Supplier 반환.
* 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수
```java
// 빈 리스트 반환
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}

// 생성자 참조를 전달
public Supplier<List<T>> supplier() {
    return ArrayList::new
}
```

#### accumulator 메서드 : 결과 컨테이너에 요소 추가하기
* 리듀싱 연산을 수행하는 함수를 반환
* 함수의 반환값은 void
```java
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}

// 메서드 참조로 변경
public BiConsumer<List<T>, T> accumulator() {
    return List::add;
} 
```

#### finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기
* 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환
* 항등 함수 반환
```java
public Function<List<T>, List<T>> finisher() {
    reutrn Function.identity();
}
```

#### combiner 메서드 : 두 결과 컨테이너 병합
* 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의
```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```
* 스트림의 리듀싱을 병렬로 수행 가능
    * 자바 7의 포크/조인 프레임워크와 Spliterator를 사용
    * 스트림의 병렬 리듀싱 수행 과정
        * 스트림을 분할해야 하는지 정의하는 조건이 거짓으로 바뀌기 전까지 원래 스트림을 재귀적으로 분할
            * 프로세싱 코어의 개수를 초과하는 병렬 작업은 호율적이지 않음
        * 모든 서브트림의 각 요소에 리듀싱 연산을 순차적으로 적용해서 서브스트림을 병렬로 처리 할 수 있음
        * 분할된 모든 서브트림의 결과를 합치면서 연산이 완료

#### Characteristics 메서드
* characteristics 메서드는 컬렉션의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환
* 스트림을 병렬로 리듀스 할 것인지 그리고 병렬로 리듀스 한다면 어떤 최적화를 선택해야 할지 힌트를 제공
* Characteristics의 열거형 세 항목
    * UNORDERED
        * 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않음
    * CONCURRENT
        * 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행 가능
        * 집합처럼 요소의 순서가 무의미한 상황에서만 병렬 리듀싱을 수행 할 수 있음
    * IDENTITY_FINISH
        * finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 생략 가능
        * 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용 가능
        * 누적자 A를 결과 R로 안전하게 형변환 가능

## 마치며
* collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터라 불리는)을 인수로 갖는 최종 연산이다.
* 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터뿐 아니라 최솟값, 최댓값, 평균값을 계싼한ㄴ 컬렉터 등이 미리 정의되어 있다.
* 미리 정의된 컬렉터인 groupingBy로 스트림의 요소를 그룹화하거나, partitioningBy로 스트림의 요소를 분할할 수 있다.
* 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
* Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발 할 수 있다.