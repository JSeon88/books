# 3. 스트림

## 3.1 외부 반복과 내부 반복

* 반복문을 사용하여 런던 출신 음악가의 수를 구하기
  * 예제 3-1 for 문을 이용
    ``` c
    int count = 0;
    for(Artist artist : allArtists) {
      if(artist.isFrom("London")) {
        count++;
      }
    }
    ```
    * 데이터 반복 할 때마다 많은 양의 관용구 코드 반드시 사용해야 함.
    * for 문이 병렬 처리되도록 코드를 작성하는 것은 상당히 어려움.
    * 반복문의 첫 문장은 반복문을 시작한다는 것만 보여줄 뿐 어떤일을 하는지는 그 안의 코드를 살펴보아야 함.
    
  * 예제 3-2 iterator를 사용한 반복문 코드 : 외부반복
    ``` java
    int count = 0;
    Iterator<Artist> iterator = allArtists.iterator();
    while(iterator.hasNext()) {
      Artist artist = iterator.next();
      if(artist.isFrom("London")) {
        count++;
      }
    }
    ```
    * 서로 다른 방식의 동작들을 한데 묶어서 추상화하기가 어려움
    * 코드가 길어짐. for문에서의 형태와 다를 바 없음.
    
  * 예제 3-3 stream을 사용한 반복문 코드 : 내부반복
    ``` java
    long count = allArtists.stream()
                             .filter(artist -> artist.isFrom("London"))
                             .count();
    ```
    * 럳던 출신의 모든 음악가 찾기 => filter
    * 찾은 음악가의 수 구하기 => count
    <br>
    
    > 스트림은 함수형 프로그래밍을 사용하여 복잡한 연산을 더욱 간결한 방법으로 컬렉션에 직접 사용할 수 있게 해주는 도구
    
## 3.2 실제로 어떻게 동작하는 것일까?

  * 예제 3-4 컬렉팅 단계없이 필터링만 하는 예제
    ``` java
    allArtists.stream()
              .filter(artist -> artist.isFrom("London"));
    ```
    > 지연적(lazy) 방식 : filter와 같은 Stream 표현을 완성하는 메소드들   
    > 즉각적(eager) 방식 :  count와 같은 함수들은 Stream 처리 후 최종값을 만드는 데 사용
    
  * 예제 3-5 지연 수행 때문에 출력되지 않는 코드
    ``` java
    allArtists.stream()
              .filter(artist -> {
                System.out.println(artist.getName());
                artist.isFrom("London");
              });
    ```
      * 출력하면 아무것도 나오지 않음
      
  * 예제 3-6 음악가 이름이 출력되는 코드
    ``` java
      allArtists.stream()
                .filter(artist -> {
                  System.out.println(artist.getName());
                  artist.isFrom("London");
                })
                .count();
     ```
      * 종료 처리를 하는 count와 같은 메소드를 가진 스트림을 추가하면 음악가 이름이 출력된다.
      
  * 연산이 즉각인지 지연적인지 어떻게 알까?
    * 해당 메소드가 무엇을 반환하는지 확인
      * Stream 반환 => 지연적 연산
      * 다른 값이나 비어 있는 값(void) 반환 => 즉각적 연산
      
  * 왜 즉각적, 지연적 방식을 구분하고 싶어할까?
    * 어떤 결과값이나 연산들이 더 필요한지 추가로 알아낼 때까지 연산을 최대한 지연시키는 것이 최종값을 만들어내는 데 더 효율적이기 때문
