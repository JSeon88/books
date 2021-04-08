# 리액티브 익스텐션

## rx.Observable 해부하기
* Observable vs Iterator
  - Observable
    * 밀어내기 방식, 언제 값을 생성할 지 스스로 정함
  - Iterator
    * 요청을 받기 위해서는 next()를 요청해야 함

> OnNext* (OnCompleted | OnError)?

- OnNext OnCompleted
  - Observable은 값 하나를 방출하고 정상적으로 종료
  - 외부 시스템을 향해 하나의 요청을 나타내고 하나의 응답을 기대할 때 이를 사용

- OnNext+ OnCompleted
  - 종료하기 전까지 여러 값을 방출
  - 데이터베이스에서 어떤 목록을 읽어와서 개별 레코드를 하나의 값으로 받아오는 과정을 나타낼 수 있음

- OnNext+
  - 무한한 이벤트 목록
  - 끝이 없기 때문에 이벤트 발생 즉시 소비해야 함

- OnCompleted 또는 OnError 중 하나만
  - OnError : 스트림 종료의 원인인 Throwable을 감싼 처리를 추가로 진행

- OnNext+ OnError
  - 스트림은 하나 이상의 이벤트를 성공적으로 방출하지만 결국 실패
  - 무한 스트림이 치명적 오류로 인해 실패하는 경우

## Observable 알림 구독

* Observable 관찰을 시작하기 위해서는 `subscribe()` 계통을 메서드를 사용
* Observable은 예외를 던지지 않음. 대신 예외를 Observable이 전파하는 다른 형태의 이벤트 알림으로 간주

```java
  tweets.subscribe(
    (Tweet tweet) -> {System.out.println(tweet),
    (Throwable t) -> {t.praintStackTrace();}}
  );
```
* 세번 째 선택 인자는 스트림 완료를 감지하는 콜백

```java
  tweets.subscribe(
    (Tweet tweet) -> {System.out.println(tweet),
    (Throwable t) -> {t.praintStackTrace();}}
    () -> {this.noMore();}
  );
```

> 언제 항목을 생성하고 언제 멈출 것인지를 RxJava가 결정하지 않는다

## Observer<T>로 모든 알림 잡아내기

> 세 가지 콜백을 위한 컨테이너로서 Observable<T>에서 발생하는 모든 알림을 받음

```java
  Observer<Tweet> observer = new Observer<Tweet>() {
    @Override
    public void onNext(Tweet tweet){
      System.out.println(tweet);
    }

    @Override
    public void onError(Throwable e){
      e.printStackTrace();
    }

    @Override
    public void onCompeted(){
      noMore();
    }
  };

  // ...

  tweets.subscribe(observer);
```

## Subscription과 Subscriber<T>로 리스너 제어하기

* Observable은 본래 여러 구독자를 보유 할 수 있음
* Observable를 구독할 수 있는 기능, 구독을 해지할 수 있는 기능을 지원하는 방법으로 Subscription, Subscriber가 있음

* 구독을 해지 하는 방법

  1. subscription 사용
    ```java
      Subscription subscription = tweets.subscribe(System.out::println);

      // ....
      subscription.unsubscribe();
    ```

  2. Subscriber<T>는 Observver<T>와 Subscription 두가지를 모두 구현하여 이벤트, 완료 혹은 실패 알람 소비와 구독 제어 모두 가능  
     ```java
      Subscriber<Tweet> subscriber = new Subscriber<Tweet>() {
        @Override
        public void onNext(Tweet tweet){
          if(tweet.getText().contatins("java")){
            unsubscribe();
          }
        }

        @Override
        public void onCompleted() {}

        @Override
        public void onError(Throwable e) {
          e.printStackTrace();
        }
      };
      tweets.subscribe(subscriber);
    ```
## Observable 만들기
* Observable.just(value)
  - 향후에 구독할 대상에 정확히 하나의 값을 방출하고 종료하는 Observable을 만듬
  - 중복 정의한(overloaded) just() 메서드로 방출할 값을 2개부터 9개까지 인자로 가능

* Observable.from(values)
  - 해당 컬랙션의 값을 방출할 수 있는 Observable을 만듬
  - Future<T>를 받는 중복 정의한 from()은 해당하는 Future가 끝나면 이벤트를 방출

* Observable.range(from, n)
  - from에서 n개의 정수값을 취해 스트림을 만듬
  - 예를 들어 range(5,3)은 5,6,7를 방출하고 종료

* Observable.empty()
  - 아무런 값도 방출하지 않고 구독을 즉시 종료

* Observable.never()
  - 알림이나 종료, 오류 이벤트 중 그 어떤 것도 방출하지 않음
  - 테스트 용도로 사용

* Observable.error()
  - 모든 구독자에서 즉시 onError() 알림을 방출
  - onCompleted()조차 방출 하지 않음

### Observable.create() 정복
```java
  Observable<Integer> ints = Observable.create(new Observable.onSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
      log("Create");
      subscriber.onNext(5);
      subscriber.onNext(6);
      subscriber.onNext(7);
      subscriber.onComplted();
      log("Completed")
    }
  });

  log("Starting"); // Observable을 생성한 것 말고 아무 일도 벌어지지 않음
  ints.subscribe(i -> log("Element: " + i));  // 구독 처리. 방출 시작
  log("Exit")

  // main: Starting
  // main: Create
  // mian: Element: 5
  // main: Element: 6
  // main: Element: 7
  // main: Completed
  // main: Exit
```

  - 구독이 클라이언트 스레드에서 발생하고 있기 때문에 결국 모든 이벤트를 받을 때까지는 usbscribe()는 클라이언트 스레드를 블록한다.

### 여러 구독자 관리하기
```java
  Observable<Integer> ints = Observable.create(subscriber -> {
    log("Create")
    subscriber.onNext(42);
    subscriber.onCompleted();
  });
  log("Starting")
  ints.subscribe(i -> log("Element A: "+i));
  ints.subscribe(i -> log("Element B: "+i));
  log("Exit")

  // main: Starting
  // main: Create
  // mian: Element A: 42
  // main: Create
  // main: Element B: 42
  // main: Exit
```
* cache()
  - create() 내부의 구독자를 각각 호출하지 않고 이미 계산된 결과를 재사용할 때 사용
  ```java
    Observable<Integer> ints = Observable.<Integer>create(subscriber -> {
      // ...
    }).cache();

    // main: Starting
    // main: Create
    // mian: Element A: 42
    // main: Element B: 42
    // main: Exit
  ```
  - 무한 스트림과 cache()를 같이 사용하면 OutOfMemoryError가 발생함을 유의해야 함

### 무한 스트림
```java
  Observable<BigInteger> naturalNumbers = Observable.create(subscriber -> {
    Runnable r = () -> {
      BigInteger i = ZERO;
      // while(true) 보다는 아래처럼 구독확인을 하는 방법을 추천.
      while(!subscriber.isInsubscribed()){
        subscriber.onNext(i);
        i = i.add(ONE);
      }
    };
    new Thread(r).start();
  });
```
* 구독 해지 시 다음 신호를 받을 때까지 기다리지 않고, 알림을 받아서 가능한 한 즉시 자원을 정리
```java
  static <T> Observable<T> delayed(T x){
    return Observable.create(subscriber -> {
      Runnable r = () -> {/* ... */};
      final Thread thread = new Thread(r);
      thread.start();
      // Thread.interrupt()를 호출하면 sleep() 안에서 InterruptedException을 발생시켜 10초간 잠든 상태를 중단 시키고 sleep()은 예외를 삼킨 뒤 깔끔하게 끝남
      subscriber.add(Subscriptions.create(thread::interrupt));
    });
  }
```
  - subscriber.isUnsubscribed()는 false를 반환하기 때문에 어떤 이벤트도 방출 안함
  - 스레드는 즉시 멈추고 어떠한 자원 낭비도 없음.

* create() 안에서 스레드를 사용하면 안되는 이유
  - 구독자는 동시에 알림을 받으면 안된다.
  - 여러 스레드에서 동시에 subscriber의 onNext() 메서드를 호출하면 안됨.
   - 복잡성을 회피하기 위해 간단하게 merge()나 flatMap() 같은 RxJava의 연산자를 쓰면 됨.

* 오류 전파
  - create() 안의 모든 표현식을 try-catch 블록으로 감싼다.
  ```java
    Observable<Data> rxLoad(int id) {
      return Observable.create(subscriber -> {
        try{
          subscriber.onNext(load(id));
          subsriber.onCompleted();
        } catch (Exception e) {
          subscriber.onError(e);
        }
      })
    }
  ```
  - fromCallable() 
    - 하나의 값으로 끝나는 Observable을 try-catch 문장으로 감싸는 구현과 비슷
    ```java
      Observable<Data> rxLoad(int id) {
        return Observable.fromCallable(() -> {
          load(id)
        });
      }
    ```

### 타이밍: timer()와 interval()
* timer()
  ```java
    Observable.timer(1, TimeUnit.SECONDS)
              .subscribe((Long zero) -> log(zero));
  ```
  - 지정된 시간만큼 지연 시킨 후 long형 0값을 방출하고 종료하는 단순한 Observable
  - 현재 스레드를 블로킹하는 대신 Observable을 만들로 subscribe()로 구독함.

* interval()
  ```java
    Observable.interval(1_000_000 / 60, MICROSECONDS)
              .subscribe((Long i) -> log(i))
  ```
  - 0부터 시작하는 연속된 long 순열을 만듬
  - range()와 달리 첫 번째 요소도 포함해 개별 이벤트를 방출하기 전에 일정한 지연 시간을 둠

### 뜨거운 Observable과 차가운 Observable
* 차가운 Observable
  - 전적으로 느긋하여 실제로 누군가 관심을 기울이지 않으면 절대 이벤트 방출을 시작하지 않음.
  - 이벤트는 느긋하게 만들어지며 어떤 식으로든 캐시 처리되지 않기 때문에 모든 구독자는 각자 별도로 스트림의 복사본을 받는다.
  - 일반적으로 Observable.create()를 써서 만들어짐
  - create() 이외에도 Observable.just()나 from(), range() 등이 있음.
  - 언제 구독하더라도 완전하고 일관된 이벤트 집합을 받음
  - 요청 할 때마다 값을 생성하며 여러 번 요청해도 되기 때문에 항목이 정확히 언제 만들어졌는지는 별로 중요하지 않음.

* 뜨거운 Observable
  - 획득한 순간 Subscriber 여부와 관계없이 즉시 이벤트 방출.
  - 심지어 아무도 구독하지 않아도 이벤트를 다운스트림으로 밀어내기 때문에 이벤트 유실이 있을 수 있음
  - Subscriber의 존재 여부가 Observable의 동작에 영향을 미치지 않으며, 서로 완전히 분리되어 독릭적임.
  - 보통 이벤트 소스를 전혀 통제할 수 없는 경우에 발생(가령 마우스 움직임이나 키보드 입력 등)
  - 처음부터 이벤트 받는다고 보장 할 수 없음
  - 일반적으로 외부에서 발생하여 오는 그대로 이벤트를 표현하는데, 이벤트를 시계열에 맞춰 늘어놓기 때문에 해당 값이 언제 발생했는지가 무척 중요함.

### rx.subjects.Subject
* AsyncSubject
  - 마지막 방출값을 기억하고 있다가 onComplete()를 호출하면 그 값을 구독자에게 보냄.
  - 완료되지 않으면 마지막 이벤트를 제외한 나머지는 무시됨

* BehaviorSubject
  - PublishSubject와 마찬가지로 구독을 시작하면 구독 이후부터 방출된 모든 이벤트를 밀어냄.
  - Subscriber는 즉시 스트림의 상태에 대한 알림을 받을 수 있음.

* ReplaySubject
  - 밀어낸 모든 이벤트의 이력을 캐싱
  - 누군가 구독하면 처음에 놓친(캐시된) 이벤트를 일괄로 받은 다음 이후 이벤트를 실시간으로 받음.
  - 스트림이 무한이거나 매우 긴 경우 위험함
    - 이러한 제약사항을 적용한 오버로딩된 ReplaySubject
      - 메모리상의 이벤트 숫자 설정(createWithSize())
      - 가장 최근의 이벤트에 대한 시간대 설정(createWithTime())
      - 혹은 createWithTimeAndSize()로 두 가지 모두에 제한 설정(둘 중 먼저 걸린 제한 적용)

* Subject에서 조심히 다루어야 하는 것들
  1. 캐시된 이벤트와 구독자 간에 공유
    - Connectable Observable을 사용할 것
  2. 동시성
    - Observer의 모든 onNext() 호출은 직렬화(순서대로)되어 있어야 하고 따라서 두 개의 스레드가 동시에 onNext()를 호출 할 수 없음
    - 그러나, Subject의 메서드를 호출하는 방식에 따라 규칙이 깨지기도 함
      - .toSerialized()를 호출함으로서 해결

## ConnectableObservable
> 여러 Subscriber를 조율하고 밑바탕의 구독 하나를 공유하는 방법<br>
Observable의 일종으로 최대 하나의 Subscriber만 유지하지만, 실질적으로는 같은 기반 리소스를 여러 Subscriber가 공유함

* 다양한 ConnectableObservable 응용
  - 중요한 부수 효과를 생성하는 경우, 혹은 심지어 '실제' Subscriber가 아직 나타나지 않았더라도 구독을 강제 할 수 있음.
  - 아무리 많은 Subscriber가 ConnectableObservable에 연결해도 생성된 Observable에 대해 단 하나의 구독만을 염.

### publish().refCount()로 구독 하나만 유지하기
* 이벤트를 밀어내는 Observable을 여러개의 Subscriber를 공유 하는데 통제할 수 없는 예제 소스
  ```java
    Observable<Status> observable = Observable.create(subscriber -> {
      System.out.println("Establishing connection");
      TwitterStream twitterStream = new TwitterStreamFactory().getInstance();
      // ....
      subscriber.add(Subscriptions.create(() -> {
        System.out.println("Discconecting");
        twitterStream.shutdowm();
      }));
      twitterStream.sample();
    });

    Subscription sub1 = observable.subscribe();
    System.out.println("Subscribed 1");
    Subscription sub2 = observable.subscribe();
    System.out.println("Subscribed 2");
    sub1.unsubscribe();
    System.out.println("Unsubscribed 1");
    sub2.unsubscribe();
    System.out.println("Unsubscribed 2");
    
    /* 
     Establishing connection
     Subscribed 1
     stablishing connection
     Subscribed 2
     Disconnecting
     Unsubscribed 1
     Disconnecting
     Unsubscribed 2
    */
  ```

* publish().refCount()를 쓴 경우
```java
  lazy = observable.publish().refCount();
  //...
  System.out.println("Before subscribers");
  Subscription sub1 = lazy.subscribe();
  System.out.println("Subscribed 1");
  Subscription sub2 = lazy.subscribe();
  System.out.println("Subscribed 2");
  sub1.unsubscribe();
  System.out.println("Unsubscribed 1");
  sub2.unsubscribe();
  System.out.println("Unsubscribed 2");

  /* 
    Before subscribers
    Establishing connection
    Subscribed 1
    Subscribed 2
    Unsubscribed 1
    Disconnecting
    Unsubscribed 2
  */
```
  - 두 번째 Subscriber가 새로운 연결을 만들지 않으며, 원본 Observable도 건드리지 않음.
  - publish().refCount() 연동은 기반 Observable을 둘러 싸서 모든 구독자를 가로챔.
    - 기본적으로 지금 이 순간 얼마나 많은 Subscriber가 있는지 셈
    - 느긋함을 유지하면서 단일 Subscriber를 공유하도록 허용할 수 있음.