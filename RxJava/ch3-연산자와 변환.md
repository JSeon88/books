# 연산자와 변환

## 핵심 연산자 : 매핑과 필터링

### 필터(filter)
  > 술어(조건)을 통해 아벤트를 계속 전달할지 버릴지를 정함

  - 연쇄 호출 방식 사용 가능(filter(p1).filter(p2))
    - 논리곱(filter(p1 && p2 && p3))으로 구현 가능
  - 변환을 재사용하거나 다른 방식으로 구성하려면(여러 개의 필터 같이) 변환을 보다 작은 단위로 나눌수록 좋음
  - 연산자를 많이 쓸수록 오베헤드를 가중하고 스택의 깊이를 더함
  - `모든 단일 연산자는 새로운 Observable을 반환하면 원본은 손대지 않고 그대로 둠`

### map()을 사용한 1:1 변환
  > map() 연산자는 원형 입력 이벤트를 사각형으로 바꾸는 함수를 받는다. <br>
  이 변환은 흘러 지나가는 개별 항목을 대상으로 적용된다.

  ```java
    Observable.just(8,9,10)
              .filter(i -> i % 3 > 0)
              .map(i -> "#" + i * 10)
              .filter(s -> s.length() < 4); 
  ```

  * 어떤값을 방출 될까?
    - map(), filter() 같은 단일 연산자는 구독하지 않는 이상 수행하지 않음
  
  ```java
    Observable.just(8,9,10)
              .doOnNext(i -> System.out.println("A: " + i))
              .filter(i -> i % 3 > 0)
              .doOnNext(i -> System.out.println("B: " + i))
              .map(i -> "#" + i * 10)
              .doOnNext(s -> System.out.println("C: " + s))
              .filter(s -> s.length() < 4)
              .subscribe(s -> System.out.println("D: " + s)); 
    
    // A: 8
    // B: 8
    // c: #80
    // D: #80
    // A: 10
    // B: 10
    // c: #100
  ```
  > doOnNext() : 흘러 지나가는 항목을 건드리지 않고 들여다볼 수 있는 연산자

### flatMap()으로 마무리하기
> map()과 비슷하지만 개별 변환한 요소를 다른(중첩된, 내부의) Observable로 반환<br>
Observable이 다른 비동기 작업을 한다면 flatMap()을 개별 업스트림 이벤트에 대한 비동기 연산으로 생성하여 분할(fork) 실행한 뒤 결과를 다시 모을 (join) 때 사용할 수 있음. <br>
`Observable<T> 와 T를 Observable<R>로 바꾸는 함수를 취함.`

* 간단한 예제
  ```java
    Observable<CarPhoto> cars() { /** **/ }
    Observable<LicensePlate> recognize(CarPhoto photo) { /** **/ }

    Observable<CarPhoto> cars = cars();

    // map 사용
    // map() 함수에서 반환하는 모든 것을 Observable 안에서 한번 더 감쌈
    Observable<Observable<LicensePlate>> plates = cars.map(this::recognize);

    //flatMap 사용
    Observable<LicensePlate> plates = cars.flatMap(this::recognize);
  ```
* flatMap()을 어떤 상황에 사용할까?
  - map()의 변환 결과가 Observable이어야 하는 경우
    - 예 : 스트림의 개별 항목이 블록되지 않고 오랫동안 수행되는 비동기 작업
  - 단일 이벤트가 여러 하위 이벤트로 확장되는 일대다 변환이 필요한 경우
    - 예 : 고객 정보 스트림이 각 고객마다 임의의 수의 주문이 가능한 주문 스트림으로 바뀜

* Iterable(List나 Set)을 반환하는 메서드를 쓰고 싶다면?
  ```java
    Observable<Order> orders = customers.map(Customer::getOrders)
                                        .flatMap(Observable::from);
    
    Observable<Order> orders = customers.flatMapIterable(Customer::getOrders);
  ```

* 이벤트 뿐만 아니라 오류와 완료 등 어떠한 알림에도 반응하는 flatMap()
  ```java
    upload(id)
            .flatMap(
                bytes -> Observable.empty(),
                e -> Observable.error(e),
                () -> rate(id)
            );
  ```
### delay() 연산자로 이벤트를 지연시키기
```java
  // timer와 flatMap을 사용
  Observable
      .timer(1, TimeUnit.SECONDS)
      .flatMap(i -> Observable.just(x,y,z))

  // delay 사용
  import java.util.concurrent.TimeInit;

  jsut(x,y,z).delay(1, TimeUnit.SECONDS)
```

* delay()와 timer()의 차이점
  - delay() : 모든 단일 이벤트를 주어진 시간만큼 미룸
  - timer() : 단순히 주어진 시간만큼 `잠들었다가` 특별한 이벤트를 방출.

```java
  import static rx.Observable.timer;
  import static java.util.concurrent.TimeUnit.SECONDS;\

  // delay 사용
  Observable
      .just("Lorem", "ipsum", "dolor", "sit", "amet", "consectetur", "adipiscing", "elit")
      .delay(word -> timer(word.length(), SECONDS))
      .subscribe(System.out::println);
  TimeUnit.SECONDS.sleep(15);

  // timer 사용
  Observable
      .just("Lorem", "ipsum", "dolor", "sit", "amet", "consectetur", "adipiscing", "elit")
      .flatMap(word -> timer(word.length(), SECONDS)
                        .map(x -> word))
```
### flatMap() 이후 이벤트 순서
