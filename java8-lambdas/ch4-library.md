# 4. 라이브러리

- 람다식 파라미터의 추론 규칙

  - 오직 하나의 대상 타입만 가능할 때에는 람다식은 함수형 인터페이스에 일치하는 인자 타입으로 추론
  - 가능한 대상 타입이 여러 개라면 이들 중 가장 명확한 타입으로 추론
  - 가능한 대상 타입이 여러개고 이들 중에 명확한 타입이 없다면, 직접 원하는 타입으로 지정

- @FunctionallInterfase 어노테이션
  - 컴파일러는 강제적으로 인터페이스가 함수형 인터페이스인지 확인
  - 어노테이션이 enum, 클래스, 또는 다른 어노테이션에 사용되었거나 타입이 하나 이상의 추상 메소드를 가지는 인터페이스라면 에러

## default method

> 자바 컴파일러에 해당 메소드를 인터페이스에 정말로 추가하고 싶다고 알려주는 것

- 디폴트 메소드와 서브 클래싱

  ```java
      public interface Parnet(){
          public void message(String body);
          public default void welcome(){
              message("Parent: Hi!");
          }
          public String getLastMessage();
      }
  ```

  ```java
      @Test
      public void parentDefaultUsed(){
          Parent parent = new ParentImpl();
          parent.welcome();
          assertEquals("Parent: Hi!", parent.getLastMessage());
      }
  ```

  - 결과값 : Parent: Hi!

  ```java
      public interface Child extends Parent{
          @Override
          public default void welcome(){
              message("Child: Hi!");
          }
      }
  ```

  - 결과값 : Child: Hi!

  ```java
      public class OverridingParent extends ParentImpl{
          @Override
          public void welcome(){
              message("Class Parent: Hi!");
          }
      }
  ```

  ```java
      @Test
      public void concreteBeastsDefault(){
          Parnet parent = new OverridingParent();
          parent.welcome();
          assertEquals("Class Parent: Hi!", parent.getLastMessage());
      }
  ```

  - 결과값 : Parent: Hi!

  ```java
      public class OverridingChild extends OverridingParent implements Child{

      }
  ```

  ```java
      @Test
      public void concreteBeatsCloserDefault(){
          Child child = new OverridingChild();
          child.welcome();
          assertEquals("Class Parent: Hi!", child.getLastMessage());
      }
  ```

  - 결과값 : Parent: Hi!

  > default 메소드보다 구체화 메소드가 우선적

## 다중 상속

    ```java
        public interface Jukebox{
            public default String rock(){
                return "...all over the wolrd!";
            }
        }
    ```
    ```java
        public interface Carriage{
            public default String rock(){
                return "...from side to side";
            }
        }

        public class MusicalCarriage implements Carriage, Jukebox{

        }
    ```

- 에러 메세지 : Class MusicalCarriage inherits unrelated defaults for rock() form types Carriage and Jukebox"

- 해결 방법

  ```java
      public class MusicalCarriage implements Carriage, Jukebox{
          @Override
          public String rock(){
              return Carriage.super.rock();
          }
      }
  ```

- 3가지 규칙
  1. 클래스는 인터페이스를 이긴다.
  2. 하위 클래스에 있는 구현이 상위 클래스의 구현보다 우선한다.
  3. 두 규칙에서 아무런 답도 찾지 못한다면 하위 클래스는 반드시 메소드를 구현하거나 abstract로 선언해야 한다.
