# RxJava를 활용한 리액티브 프로그래밍

## 리액티브 프로그래밍과 RxJava

* 리액티브 프로그래밍
  > 데이터나 이벤트 변화의 반응에 초첨을 맞춘 프로그래밍<br>
  `비동기, 이벤트 기반 프로그램을 작성하기 위한 라이브러리`

* 리액티브 함수형 프로그래밍
  > 프로그래밍에 대한 하나의 접근 방식, 즉 명령형 시스템상의 추상화. <br>
  예를 들어 네트워크나 스레드 경계에서 상태들의 복잡한 상호작용을 명령형으로 정의 하지 않아도 됨. <br>
  즉, `동시성과 병렬성 해결`
* RxJava를 사용한 리액티브 프로그래밍
  > 함수형 프로그래밍의 영향을 바탕으로 리액티브 명령형 방식의 전형적인 위험 요소를 회피하기 위한 `선언적 접근` 사용

## 언제 리액티브 프로그래밍이 필요한가

* 마우스 움직임이나 클릭, 키보드 타이핑, GPS 신호, 터치 이벤트 등을 처리 할때
* 비동기성을 띠는 디스크나 네트워크 등 지연 바인딩 I/O 이벤트 응답
* 서버의 시스템, 이벤트나 앞서 나온 사용자 이벤트, 하드웨어 신호 등 통제 불가능한 애플리케이션에 발생하는 이벤트나 데이터를 다룰 때

## RxJava는 어떻게 동작하는가
### RxJava의 핵심
  * Observable 타입
    - 비동기 시스템의 흐름 제어나 배압(backpressure)에 대한 접근 방식
    - 비동기 피드백 채널 지원
  * 밀어내기(reactive) 방식 지향, 끌어오기(interactive) 방식도 사용 가능
  * 즉시 동작하지 않고 지연 실행
  * 비동기와 동기 방식 모두 사용 가능
  * 시간에 따라 0,1,다수 혹은 무한 개를 아우르는 이벤트를 다룰 수 있음
#### 밀어내기와 끌어오기
  * 밀어내기를 통한 이벤트 수신
    - Observable/Observer 쌍을 구독으로 연결
    ```java
      interface Observable<T> {
        Subscription subscribe(Observer s)
      }
    ``` 
    - Observer는 구독을 통해 3가지 유형의 이벤트를 받음
      ```java
        interface Observer<T> {
          // onNext 함수를 통한 데이터
          void onNext(T t)
          // onError 함수를 통한 오류
          void onError(Throwable t)
          // onCompleted 함수를 통한 스트림 완료 통보
          void onCompleted()
        }
      ```
      * onNext : 전혀 호출되지 않거나 한번에 여러번, 혹은 무한히 호출될 수 있음
      * onError, onCompleted : 종료 이벤트, 둘 중 하나만 단 한번 호출됨
    - Subscriber : 조금 더 발전된 Observer
      ```java
        interface Subscriber<T> implements Observer<T>, Subscription {
          void onNest(T t)
          void onError(Throwable t)
          void onCompleted()
          ...
          // Observable 스트림 구독을 끊을 때 사용
          void unsubscribe()
          // 흐름 제어 시 생산자와 소비자 간의 양방향 소통 채널을 구성할 때 사용
          void setProducer(Producer p)
        }
      ```
#### 비동기와 동기
  * Observable의 동작의 기본값은 동기 방식.
  * 일반적으로는 비동기 방식으로 동작함.
  * Observable을 동기 방식으로 구독하는 경우라면
    - 모든 데이터를 구독자 스레드에서 방출하고 종료
    - 블로킹 네트워크 I/O 라면
      * Observable은 구독 스레드를 동기적으로 블로킹
      * 네트워크 I/O의 블로킹이 해제되면 onNext를 실행
    ```java
      Observable.create(s -> {
        s.onNext("Hello World!")
        s.onComplete()
      }).subscribe(hello -> System.out.println(hello));
    ```
  * Observable 이벤트 생성의 중요한 기준
    - `블로킹/논블로킹` 여부
  
#### 메모리 내부 데이터
  * 메모리에 값이 잇으면 동기적으로 값을 발송
  * 그렇지 않으면, 비동기로 네트워크 호출을 한 다음 값을 받았을 때 반환

#### 동기 방식 계산(예: 연산자)
  * 동기 방식을 유지 하는 이유
    - 스트림 조합과 연산자를 통한 변환 때문
  * 대부분의 연산자는 동기 방식
  * 대부분의 Observable 함수 파이프라인이(비동기 방식이어야 하는 timeout, observeOn 등을 제외하고) 동기 방식. Observable 자체는 비동기 방식일 수 있음
  * 동기와 비동기 방식의 혼합 예제
    ```java
      Observable.create(s -> {
        // 비동기 구독과 데이터 방출
      })
      .doOnNext(i -> System.out.println(Thread.currentThread()))
      .filter(i -> i%2 == 0)
      .map(i -> "값" + i + " 는 " + Thread.currentThread() + " 에서 처리된다")
      .subscribe(s -> System.out.println("값 =>" + s));
      System.out.println("값이 출력되기 전에 나온다")
    ```
    - Observable은 비동기
    - subscribe는 논블로킹
    - filater, map 연산자는 동기적으로 실행
> Observable는 동기/비동기 방식을 모두 지원

#### 동시성과 병렬성
* 단일 Observable 스트림은 `동시성이나 병렬성 둘 다 허용 안함.`
> 병렬성(parallelism) : 동시에 수행하는 작업들 <br>
동시성(concurrency) : 여러 작업들을 합성하거나 번갈아(interleaving) 수행 
* onNext, onCompleted, onError 이벤트는 스레드 안전. 동시에 방출되지 않는다.
* 동시성과 병렬성의 장점을 취할려면 어떻게 해야 하는가?
  - 하나의 Observable 스트림은 항상 직렬화
  - 각각의 Observable 스트림을 서로 독립적으로 조작 할 수 있어 동시에 병렬 수행 할 수 있음.
  > 즉, 비동기 스트림을 모아 동시에 수해하는 merge, flatMap 을 사용하면 됨.
  ```java
    Observable<String> a = Observable.create(s -> {
      new Thread(( -> {
        s.onNext("one");
        s.onNext("two");
        s.onCompleted()
      })).start();
    });
    
    Observable<String> b = Observable.create(s -> {
      new Thread(() -> {
        s.onNext("three")
        s.onNext("foru")
        s.onComplted()
      }).start();
    })

    // 동시에 a와 b를 구독하여 제 3의 순차적인 스트림으로 병합한다
    Observable<String> c = Observable.merge(a,b);
  ```
* 어째서 onNext를 동시에 호출할 수 있게 만들지 않았을까?
  1. 굳이 필요 없는 경우에도 모든 Observer에 동시 호출에 대한 방어 코드를 추가 해야 했음
  2. 몇몇 연산자는 동시 방출이 불가능
  3. 대부분의 데이터가 순차적으로 도달하더라도 모든 옵저버와 연산자가 스레드로부터 안전하게 보호되어야 하기 때문에 성능이 동기화 오버헤드에 영향을 받음

#### 느긋함과 조급함
> Observable은 느긋하고 구독하지 않는 한 시작하지 않는다

* 생성이 아니라 구독이 작업을 시작한다
  - Observable 객체 생성이 어떤 작업을 유발하지 않음.
* Observable은 재사용 할 수 있다.

#### 쌍대성(duality)
| 끌어오기(Iterable)  | 밀어내기(Observable)  |
| ----------------- | ------------------  |
| T next()          | onNext(T)           |
| throws Exception  | onError(Throwable)  |
| returns           | onCompleted()       |
* Stream에서 Iterable이나 List를 통해 동기화 끌어오기를 사용한 것처럼 Rx Observable은 밀어내기를 통한 비동기 데이터 프로그래밍을 할 수 있도록 함.

#### 카디널리티(Cardinality)
* Observable은 여러개의 값을 비동기적으로 밀어낼 수 있음

|               | 한 개                 | 여러 개                    |
| ------------- | -------------------- | -----------------------  |
| Synchronous   | T getData()          | Iterable<T> getData()    |
| Asynchronous  | Future<T> getData()  | Observable<T> getData()  |

* 어째서 Future 대신 Observable이 쓸모 있을까?
  - 이벤트 스트림 또는 다중값 응답을 다룰 수 있기 때문

  - 이벤트 스트림
    ```java
      // Observable

      //생산자
      Observable<Event> mouseEvents = .....;
      //소비자
      mouseEvents.subscribe(e -> doSomethingWithEvent(e));

      // Futrue

      //생상자
      Future<Event> mouseEvents = ......;
      //소비자
      mouseEvents.onSuccess(e -> doSomethingWithEvent(e));
    ```
    * 아래와 같은 경우엔 Observable이 더 이점이 있음.
      - 소비자가 이벤트를 뽑아낼 필요가 있는가?
      - 생산자가 이벤트를 대기열에 준비해 두는가?
      - 각각의 이벤트를 가져오는 도중에 유실 될 수도 있는가?
  
  - 다중 값
    * 목록이 크거나 다양한 원격지의 데이터 소스를 끌어와 목록 요소를 채워야 한다면 Observable이 `성능이나 반응 시간` 측면에서 이점이 있음.
      - 컬렉션 전체를 기다리지 않고 항목을 받는 대로 처리할 수 있기 때문
    
  - 구성(Composition)
    * Future도 단일 응답을 모아 값 하나를 갖는 다른 Future를 방출 가능
      - 모든 Future가 완료될 때까지 대기
    * Observable는 merge를 통해 준비가 대는 대로 즉시 방출 가능

  - Single
    * API를 설계하거나 사용할 때는 단일 값 표현이 단순해서 좋음
    * 이런 이유로, 느긋한 Future와 동격인 Single 형을 제공
    * 장점
      1. 느긋한 특성으로 인해 여러 번 구독할 수 있으며, 쉽게 합성 가능
      2. RxJava API와 잘 맞아 Observable과 손쉽게 상호 작용 가능
      ```java
        public static Single<String> getDataA() {
          return Signle.<String> create(o -> {
            o.onSuccess("DataA")
          }).subscribeOn(Schedulers.io());
        }

        public static Single<String> getDataB() {
          return Single.just("DataB").subscribeOn(Schedulers.io());
        }

        // a와 b를 2개의 값을 가진 Observable 스트림으로 병합
        Observable<String> a_merge_b = getDataA().mergeWith(getDataB());
      ```
      - 무엇이 먼저 끝나느냐에 따라 방출 순서는 [A,B] 혹ㄷ은 [B,A]일 수 있음
    * Observable 대신 Single을 써서 `값 하나 짜리 스트림`을 나타낼 수 있음
    * Observabler과 Single의 동작에 대한 고려상황
      - Single
        * 오류 응답
        * 응답 없음
        * 정상 응답
      - Observable
        * 오류 응답
        * 응답 없음
        * 값이 없는 정상 응답 후 종료
        * 단일 값 정상 응답 후 종료
        * 다중 값 정상 응답 후 종료
        * 하나 이상의 정상 응답 후 종료하지 않음(추가적인 값을 기다리며 대기)
  
  - Completable
    * 반환형이 없을 때 사용
    ```java
      Completable c = writeToDatabase("data");
    ```
    * 보편적인 사용례
      - 비동기적인 쓰기 작업
      - 반환 값은 필요 없지만 성공이나 실패 여부가 필요할 때
  
  - 0에서 무한대까지
    |               | 0개                         | 한 개                 | 여러 개                    |
    | ------------- | --------------------------- | -------------------- | -----------------------  |
    | Synchronous   | void doSomething()          | T getData()          | Iterable<T> getData()    |
    | ASynchronous  | Completable doSomething()   | Future<T> getData()  | Observable<T> getData()  |

