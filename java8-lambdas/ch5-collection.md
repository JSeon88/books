# 5. 고급 컬렉션과 컬렉터

## 메소드 참조

> Classname::methodName

```java
// 기존 메소드 사용
artist -> artist.getName()
// 메소드 참조 사용
Artist::getName
```

- 커스텀 클래스 생성

```java
Artist::new
```

- String 배열 생성

```java
Strkng[]::new
```

## 엘리먼트 정렬

- 인카운터 정렬을 의도적으로 만든 코드

```java
Set<Integer> numbers = new HashSet<>(asList(4,3,2,1));
List<Integer> sameOrder = numbers.stream()
                                 .sorted()
                                 .collect(toList());

assertEquals(asList(1,2,3,4), sameOrder);
```

- forEach
  - 인카운터 정렬 보장하지 않음
  - 정렬 순서 보장하고 시다면 forEachOrdered

## 컬렉터

> 스트림으로부터 다양한 값을 생성하기 위한 목적을 가진 생성자

- 사용자 정의 컬렉션 만들기

```java
stream.collect(toCollection(TreeSet::new));
```

- 컬렉터를 사용하여 단일 값 만들기

  - maxBy, minBy

  ```java
  public Optional<Artist> biggersGroup(Stream<Artis> artists) {
      Function<Artist, Long> getCount = artist -> artist.getMembers().count();
      return artists.collect(maxBy(comparing(getCount)));
  }
  ```

  - comparing

    > 인자 : Function<T,R> <br>
    > return : Comparator <br> <br>
    > test.stream().sorted(comparing(e -> e.getName()));

  - thenComparing : 2차 정렬

  ```java
  Function<Persion, Integer> firstSort = person -> peson.getAge();
  Function<Persion, String> secondSort = person -> person.getName();

  people.stream().sorted((firstSort).thenComparing(secondSort))
  ```

  - averagingInt : 평균

  ```java
  public double averageNumberOfTracks(List<Album> albums){
      return albums.stream()
                   .collect(averagingInt(album -> album.getTrackList().size()));
  }
  ```

  - summingInt : 값들의 합
  - SummaryStatistics : count, min, max, sum, average 등의 statistics 정보를 계산해주는 클래스

- partitioningBy : 데이터 파티셔닝

  > 주어진 조건으로 엘리먼트가 true 그룹인지 false 그룹인지 알아낸다. 그리고 조건에 의해 나누어진 List를 가지는 Map을 반환한다.

  ```java
    pulbic Map<Boolean, List<Artist>> bandsAndSolo(Stream<Artist> artists){
        return artists.collect(partitioningBy(Artist::isSolo));
    }
  ```

- groupingBy : 데이터 그룹화
