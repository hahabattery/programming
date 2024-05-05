---
layout: default
title: Exceptions
parent: Java
nav_order: 10
---

# Exception

 * Java Exception best practice
   * https://soft.plusblog.co.kr/160
   * https://www.javacodegeeks.com/10-best-practices-to-handle-java-exceptions.html

 * exception-handling-in-spring-mvc
   * https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc


체크 예외와 언체크 예외의 차이는 예외를 처리할 수 없을 때 예외를 밖으로 던지는 부분에 있다. 이 부분을 필수로 선언
해야 하는가 생략할 수 있는가의 차이다.

### Checked Exception
 * Checked Exception을 compile-time exception이라고 한다.

 * 컴파일 단계에서 체크

 * 예외 처리 필수
   + => try catch 로 잡아서 예외를 처리하거나 상위 메소드로 넘겨줘야함

 * Spring에서 DB Transaction 기본 롤백 대상이 아니라서 롤백 처리하려면 추가 처리 필요
   + @Transactional 의 rollbackFor 옵션 참고

> In its default configuration, the Spring Framework’s transaction infrastructure code only marks a transaction for rollback in the case of runtime, unchecked exceptions; that is, when the thrown exception is an instance or subclass of RuntimeException. ( Errors will also - by default - result in a rollback). Checked exceptions that are thrown from a transactional method do not result in rollback in the default configuration.


 * 예)
   + SQLException
   + ClassNotFoundException

### Unchecked Exception
 * Unchecked Exception을 runtime exception이라고 한다.

 * 예외 처리 필수 아님
   + => 예측할 수 없는 예외라서 필수 처리 불가능

 * Spring에서 DB Transaction 롤백 대상
   + @Transactional rollbackFor 에 기본 옵션으로 들어가있기 때문에 예외 발생 시 롤백 처리됨

 * 예)
   + NullPointerException
   + IllegalArgumentException


### 예외 클래스 구조
![](/images/java/Exception-Class.png)
 * 모든 예외 클래스는 Throwable 클래스를 상속받고 있으며, Throwable은 최상 클래스 Object의 자식 클래스이다.
 * Throwable을 상속받는 클래스는 Error와 Exception이 있다.
 * Error는 시스템 레벨의 심각한 수준의 에러이기 때문에 시스템에 변화를 주어서 문제를 처리해야하는 경우가 일반적이다.
 * Exception은 개발자가 로직을 추가하여 처리할 수 있다.

 * Exception의 자식 클래스 중에서 RuntimeException을 제외한 모든 클래스는 CheckedException이며, RuntimeException과 그의 자식 클래스들을 Unchecked Exception이라 한다.

### 예외의 종류와 차이점
![](/images/java/checked-unchecked-exception.png)
 * 위 표에서 기본적인 예외를 비교하였다. 꼭 알고 있어야 하는 것이 예외 발생시의 트랜잭션 roll-back여부이다.
 * 기본적으로 Checked Exception은 예외가 발생하면 트랜잭션을 roll-back하지 않고 예외를 던져준다. 하지만 Unchecked Exception은 예외 발생시 트랜잭션을 roll-back한다.
 * 트랜잭션의 전파방식 즉, 어떻게 묶어놓느냐에 따라서 Checked Exception이냐 Unchecked Exception이냐의 영향도가 크다.
 * roll-back이 되는 범위가 달라지기 때문에 개발자가 이를 인지하지 못하면, 실행결과가 맞지 않거나 예상치 못한 예외가 발생할 수 있다.
 * 그래서 반드시 이를 인지하고 트랜잭션을 적용시킬 때 전파방식(propagation behavior)와 롤백규칙를 파악해야 한다.

### 예외 처리 방법
![](/images/java/exception-handling.png)

##### 예외 복구
```java
int maxretry = MAX_RETRY;
while(maxretry-- > 0) {
  try {
    // 예외가 발생할 가능성이 있는 시도
	return; // 작업성공시 리턴
  } catch (SomeException e) {
    // 로그 출력. 정해진 시간만큼 대기
  } finally {
    // 리소스 반납및 정리 작업
  }
}
throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```

 * 예외 복구의 핵심은 예외가 발생하여도 애플리케이션은 정상적인 프름으로 진행된다는 것이다. 위 코드는 예외를 복구하는 코드이다.
 * 이 예제는 네트워크가 환경이 좋지 않아서 서버에 접속이 안되는 상황의 시스템에 적용하면 효율적이다. 예외가 발생하면 그 예외를 잡아서 일정 시간만큼
 * 대키하고 다시 재시도를 반복한다. 그리고 최대 재시도 횟수를 넘기면 예외를 발생시킨다. 재시도를 통해 정상적인 르픔을 타게 한다거나, 예외가 발생하면
 * 이를 미리 예측하ㅕ 다른 흐름으로 유도시키도록 구현하면 비록 예외가 발생하였어도 정상적으로 작업을 종료할 수 있을 것이다.

##### 예외처리 회피
```java
public void add() throws SQLException {
   ... // 구현 로직
}
```
 * 예외가 발생하면 throws를 통해 호출한 쪽으로 예외를 던지고 그 처리를 회피하는 것이다. 호출한 쪽에서 다시 예외를 받아 처리하도록하거나,
 * 해당 메소드에서 이 예외를 던지는 것이 좋을 때만 사용할 것

 * Runtime 예외와 같이 강제로 잡아주지 않아도 되는 예외의 경우는 회피시 특별한 처리를 해주지 않아도 되지만,
 * 그외의 Checked Exception(Compile Exception) 같이 강제로 예외처리가 필요한 예외를 발생시킬 때에는 try-catch 문을 사용하든지 throws 키워드를 통해 밖으로 던져주어야 한다. 

 * 그러니까 unchecked exception(Runtime Exception)의 경우 throws는 아무런 의미도 가지지 않습니다. throws는 checked exception의 처리 방법.

 * 보통의 경우 다음과 같이 throws 를 통해 발생시킨 예외를 밖으로 던짐과 동시에 '현재 메서드를 사용하면 이와 같은 예외가 발생한다' 라는 발생 조건을 알려준다. 
```java
public static void generateException() throws RuntimeException{
    throw new RuntimeException();
}

public static void generateException() throws Exception{
    throw new Exception();
}
```

##### 예외 전환
```java
catch(SQLException e) {
  ...
  throw DuplicateUserIdException();
}
```
 * 예외를 잡아서 다른 예외를 던진다. 호출한 쪽에서 예외를 받아서 처리할 때 좀더 명확하게 인지할 수 있도록 돕기위한 방법.
 * 어떤 예외인지 명확해야 처리가 수월해지기 때문인데, 예를 들어 Checked Exception중 복구가 불가능한 예외가 잡혔다면
 * 이를 Unchecked Exception으로 전환하여서 다른 계층에서 일일이 예외를 선언할 필요가 없도록 할 수도 있다.



# Exception과 DB 트랜잭션

보통 Checked Exception 이면 롤백하지 않고, Unchecked Exception 이면 롤백한다고 알고 있는데, 이것은 Spring의 기본동작이다.

Spring의 Transaction 처리에서 기본 동작이 Checked Exception이면 롤백하지 않고, Unchecked Exception이면 롤백한다.
하지만, Spring에서도 이러한 설정을 쉽게 변경할 수 있도록 해주기 때문에, Java 자체는 이런 규칙은 따로 존재하지 않는다고 볼 수 있다.




# java와 c의 비교 (예외처리의 필요성)
예외처리에 대한 메커니즘이 없으면 'Return Code'를 이용해 예외 상황에 대한 정보를 상위 컨텍스트로 넘겨줘서 처리해야한다. 대표적으로 C언어로 작성된 코드가 이런 방식을 사용한다.
사족을 달자면, C언어에서 조차 setjmp(), longjmp(), Recovery Information을 전역 변수 혹은 Shared Memory 등에 만들어 놓고 TRY-CATCH 매크로를 만들어 사용하는 경우가 많다.
프로그램의 실행과정에서 발생하는 이상상황을 에러코드로 처리하면 간단하다. 프로그램의 실행 컨텍스트가 중간에 끊기는 일도 없다.
하지만 함수를 사용하는 쪽 코드에서 항상 리턴코드를 체크하고 에러 상황에 대한 동작을 작성해놔야 한다. 이렇게 되면 예외상황에 대한 처리 코드가 덕지덕지 붙게되어 코드의 가독성이 떨어지고, 단순한 로직을 구현함에도 복잡한 예외 상황에 대한 방어코드가 많아지는 현상을 경험하게 된다.
또한, 단순히 리턴코드로만 에러를 전파하는 경우 에러 발생의 컨텍스트를 알 수 없다는 단점이 있다. 예를 들어 파일을 열수 없다는 에러코드가 리턴되었을 때, '어떤 파일이 없는데???'라는 의문에는 답을 할 수가 없게됨.

리턴코드를 사용하는 C언어 방식과 다르게 자바에서는 예외가 발생했을 때 프로그램의 실행 컨텍스트가 인터럽트(Interrupt) 되낟. 당장 중단되고 예외가 발생하여 콜스택을 따라 상위로 예외가 던져진다. 발생한 예외를 처리하는 catch 절이 없다면 프로그램은 결국 종료된다.
예외는 객체 형태로 다양한 정보를 담아 상위 컨텍스트로 던져진다. 예외를 처리하는 쪽에서는 예외 객체를 통해 예외처리에 필요한 정보를 얻어 낼 수 있다. 파일이 없다는 예외가 발생한 경우, 어떤 파일이 없는지는 예외 객체를 통해 알 수 있다.
