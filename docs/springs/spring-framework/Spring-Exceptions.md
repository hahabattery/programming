---
layout: default
title: Spring Exceptions
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---


# 참조

https://jaehun2841.github.io/2018/08/30/2018-08-25-spring-mvc-handle-exception/#%EB%93%A4%EC%96%B4%EA%B0%80%EB%A9%B0

https://mflash.dev/post/2020/07/26/error-handling-for-a-spring-based-rest-api/

https://develop-writing.tistory.com/102


 * Java Exception best practice
   * https://soft.plusblog.co.kr/160
   * https://www.javacodegeeks.com/10-best-practices-to-handle-java-exceptions.html

 * exception-handling-in-spring-mvc
   * https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc



---
# Exception

### Checked Exception
 * Checked Exception을 compile-time exception이라고 한다.

 * 예외 처리 필수
   + => try catch 로 잡아서 예외를 처리하거나 상위 메소드로 넘겨줘야함

 * Spring에서 DB Transaction 기본 롤백 대상이 아니라서 롤백 처리하려면 추가 처리 필요
   + @Transactional 의 rollbackFor 옵션 참고

> In its default configuration, the Spring Framework’s transaction infrastructure code only marks a transaction for rollback in the case of runtime, unchecked exceptions; that is, when the thrown exception is an instance or subclass of RuntimeException. ( Errors will also - by default - result in a rollback). Checked exceptions that are thrown from a transactional method do not result in rollback in the default configuration.

 * 컴파일 단계에서 체크

 * 예)
   + SQLException
   + ClassNotFoundException


https://bcp0109.tistory.com/303 하단의 댓글중 ...
>
>1.2. Transaction Rollback
>DB 처리 도중 RuntimeException 이 발생하면 롤백이 진행되지만 Checked Exception 은 발생해도 데이터가 롤백되지 않고 커밋까지 완료됩니다.
>만약 예외 발생 시 롤백을 진행하고 싶다면 try catch 로 잡아서 Uncheked Exception 을 던져줘야 합니다.
>
>허허.
>
>왜 굳이 귀찮게 try-catch 로 성가신 작업을 하고 있나?
>
>rollBackFor 속성으로 해당 메서드나 서비스 레벨에서 롤백 정책을 바꿔주면 되지
>
>스프링이 기본적으로 롤백을 관리하는 예외 타입은 Unchecked Exception 까지니까
>Checked Exception 이 발생 했을 때 롤백 처리 하려고 try/catch 로 묶고 롤백 적용되려고 바꿔 던지려는 의도란 건 알겠다만,
>
>원하는 특정 서비스나 메서드 레벨에서
>롤백 정책을 Checked Exception 까지 지원되도록 바꿔주면 될 것 아닌가?
>
>그럼 번거롭게 try/catch 쓰면서 조잡스럽게 예외 바꿔치기 할 필요가 없다.
>
>단순 롤백 처리 하려고 마치 옛날 JSP/Servlet/Model2 에서 롤백 처리 하듯
>try/catch 로 묶고 조잡하게 예외 타입을 Unchecked Exception 으로 바꿔치기 해서 던지는 건
>아주 스.알.못 같은 비효율적 방식이다.
>마치 스프링에서 Unchecked Exception 만 롤백 처리된다는 것이 '불변의 고정된 정책'인 냥
>'스프링님께서 정해주신 불변의 LAW를 어떻게든 지키겠습니다 롤백 처리 해주십쇼 낑낑!!' 꼴과 다름 없다.
>
>프레임워크를 사용하고 있으면서 프레임워크의 장점은 사용할 줄 모르는 롤백 처리지.
>
>그럼 프레임워크의 장점을 이용한다면?
>'스프링 니가 롤백을 위해 해줄 수 있는 METHOD 들을 내놔봐' 라고 물어볼 것이고
>아주 간편하고 유연하게 롤백 정책과 적용 범위를 변경할 수 있다.
​>
>스프링의 기본적인 롤백 적용 예외타입은 Unchecked Exception 이지만
>@Transaction의 rollbackFor 속성을 이용해,
>원하는 서비스의 클래스 레벨 or 메소드 레벨에서 롤백할 예외 타입을 아주 편하게 설정할 수 있다
>
>@Transaction의 rollbackFor 이 속성을 모른다면 알아보고 정리에 추가하시게
>



### Unchecked Exception
 * Unchecked Exception을 runtime exception이라고 한다.

 * 예외 처리 필수 아님
   + => 예측할 수 없는 예외라서 필수 처리 불가능

 * Spring에서 DB Transaction 롤백 대상
   + @Transactional rollbackFor 에 기본 옵션으로 들어가있기 때문에 예외 발생 시 롤백 처리됨

 * 예)
   + NullPointerException
   + IllegalArgumentException




---
# 예외처리
Java의 기본적인 예외처리는 Java 내용을 확인할 것
Spring에서의 예외처리시에 사용하는 어노테이션은 아래와 같다.


##### @ControllerAdvice
ViewResolver를 통해서 예외 처리 페이지로 리다이렉트 시키려면 @ControllerAdvice만 써도 되고, API서버여서 에러 응답으로 객체를 리턴해야한다면 @ResponseBody 어노테이션이 추가된 @RestControllerAdvice를 적용하면 되는 것이다

##### @RestControllerAdvice
@ControllerAdvice와 동일한 역할 즉, 예외를 잡아 핸들링 할 수 있도록 하는 기능을 수행하면서 @ResponseBody를 통해 객체를 리턴할 수도 있다는 얘기다.

@RestController에서 예외가 발생하든 @Controller에서 예외가 발생하든 @ControllerAdvice + @ExceptionHandler 조합으로 다 캐치할 수 있고 @ResponseBody의 필요 여부에 따라 적용하면 된다

 * 만약에 전역의 예외를 잡긴하되 패키지 단위로 제한할 수도있다.
 ```java
 @RestControllerAdvice("com.example.demo.login.controller")
 ```

---
# @ExceptionHandler (컨트롤러단에서 처리)
@ExceptionHandler는 Controller계층에서 발생하는 에러를 잡아서 메서드로 처리해주는 기능이다.

Service, Repository에서 발생하는 에러는 제외한다.

```
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```
이렇게 @Controller로 선언된 클래스 안에서 @ExceptionHandler 어노테이션으로 메서드 안에서 발생할 수 있는 에러를 처리할 수 있다.



```
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler({FileSystemException.class, RemoteException.class})
    public ResponseEntity<String> handle(Exception ex) {
        // ...
    }
}
```
여러개의 Exception을 처리하는 예.

>We generally recommend that you be as specific as possible in the argument signature, reducing the potential for mismatches between root and cause exception types. Consider breaking a multi-matching method into individual @ExceptionHandler methods, each matching a single specific exception type through its signature.



---
# @ResponseStatus... 들

Spring의 HTTP Status 응답 처리

### @ResponseStatus

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "Data Not Found")
public class DataNotFoundException extends RuntimeException { }
```

Spring 3 부터 등장.

@ResponseStatus은 HTTP Status 와 Response 정보를 생성해줌.

개발자가 정의한 Exception 이 발생하면 해당 Status 와 Message 를 전달합니다.

테스트를 위해 Controller 에서 강제로 Exception 을 발생 시켜보겠습니다.


 * @ResponseStatus 없이 그냥 NotFoundException("No Data")
```
{
  "timestamp": "2021-03-13T07:23:00.732+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "No Data",
  "path": "/auth/signup"
}
```


 * @ResponseStatus 정의된 Custom Exception 사용
```
{
  "timestamp": "2021-03-13T07:30:38.299+00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Data Not Found",
  "path": "/auth/signup"
}
```


**문제점**

@ResponseStatus 에 정의한 대로 잘 나오는 걸 확인할 수 있습니다.

이 방법은 별다른 설정 없이 어노테이션 추가만으로 간단하게 Custom Exception 을 만들 수 있지만, 한 가지 단점이 있습니다.

위에서 예시로 데이터가 없는 경우 DataNotFoundException 를 리턴하게 했는데, 이 Exception 은 항상 동일한 HTTP Status 와 Message 를 리턴합니다.

같은 Exception 이 발생하는 상황이더라도 다른 Message 를 보내는게 불가능합니다.


### @ResponseStatusException

```
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No Data");
```

Spring 5 부터 제공하는 ResponseStatusException 의 일반적인 사용입니다.

생성자로 HTTP Status 와 String 만을 받습니다.

만약 없는 데이터의 종류가 다르다면 ?? No Member Data, No Profile Data 등등.. 이렇게 표현해야 합니다.

**문제점**

하지만 이렇게 명확한 규격 없이 String 으로만 받으면 여러 사람들이 작업할 때 중복된 응답을 주거나 이미 있는 응답을 새로 만들거나, 오타로 인해 실수할 가능성도 있습니다.



---
# Spring에서의 예외처리 ( 더 나은 방법)

https://bcp0109.tistory.com/303  <= 여기 글을 정리한 내용

[Error Handling for REST with Spring](https://www.baeldung.com/exception-handling-for-rest-with-spring)

[Spring ResponseStatusException](https://www.baeldung.com/spring-response-status-exception)

[Error handling for a Spring-based REST API](https://mflash.dev/post/2020/07/26/error-handling-for-a-spring-based-rest-api/)


Spring 에는 프로젝트 전역에서 발생하는 Exception 을 한 곳에서 처리할 수 있다.

Enum 클래스로 ErrorCode 를 정의하면 Exception 클래스를 매번 생성하지 않아도 된다.

실제 클라에게 날라가는 응답에서 code 부분만 확인하면 어떤 에러가 발생했는지 쉽게 파악 가능하다.



 * ErrorCode : 핵심. 모든 예외 케이스를 이곳에서 관리함
 * CustomException : 기본적으로 제공되는 Exception 외에 사용
 * ErrorResponse : 사용자에게 JSON 형식으로 보여주기 위해 에러 응답 형식 지정
 * GlobalExceptionHandler : Custom Exception Handler
   + @ControllerAdvice : 프로젝트 전역에서 발생하는 Exception 을 잡기 위한 클래스
   + @ExceptionHandler : 특정 Exception 을 지정해서 별도로 처리해줌

### Error 관련 properties 설정

```yml
# application.yml
server:
  error:
    include-exception: false      # Response 에 Exception 을 표시할지
    include-message: always       # Response 에 Exception Message 를 표시할지 (never | always | on_param)
    include-stacktrace: on_param  # Response 에 Stack Trace 를 표시할지 (never | always | on_param) on_trace_params 은 deprecated
    whitelabel.enabled: true      # 에러 발생 시 Spring 기본 에러 페이지 노출 여부
```


### ErrorCode

```java
@Getter
@AllArgsConstructor
public enum ErrorCode {

    /* 400 BAD_REQUEST : 잘못된 요청 */
    INVALID_REFRESH_TOKEN(BAD_REQUEST, "리프레시 토큰이 유효하지 않습니다"),
    MISMATCH_REFRESH_TOKEN(BAD_REQUEST, "리프레시 토큰의 유저 정보가 일치하지 않습니다"),
    CANNOT_FOLLOW_MYSELF(BAD_REQUEST, "자기 자신은 팔로우 할 수 없습니다"),

    /* 401 UNAUTHORIZED : 인증되지 않은 사용자 */
    INVALID_AUTH_TOKEN(UNAUTHORIZED, "권한 정보가 없는 토큰입니다"),
    UNAUTHORIZED_MEMBER(UNAUTHORIZED, "현재 내 계정 정보가 존재하지 않습니다"),

    /* 404 NOT_FOUND : Resource 를 찾을 수 없음 */
    MEMBER_NOT_FOUND(NOT_FOUND, "해당 유저 정보를 찾을 수 없습니다"),
    REFRESH_TOKEN_NOT_FOUND(NOT_FOUND, "로그아웃 된 사용자입니다"),
    NOT_FOLLOW(NOT_FOUND, "팔로우 중이지 않습니다"),

    /* 409 CONFLICT : Resource 의 현재 상태와 충돌. 보통 중복된 데이터 존재 */
    DUPLICATE_RESOURCE(CONFLICT, "데이터가 이미 존재합니다"),

    ;

    private final HttpStatus httpStatus;
    private final String detail;
}
```

 * 에러 형식을 Enum 클래스로 정의합니다.
 * 응답으로 내보낼 HttpStatus 와 에러 메세지로 사용할 String 을 갖고 있습니다.
 * ResponseStatusException 과 비슷해 보입니다. 하지만 가장 큰 차이점은 개발자가 정의한 새로운 Exception 을 모두 한 곳에서 관리하고 재사용 할 수 있다는 점입니다.


### CustomException

```
@Getter
@AllArgsConstructor
public class CustomException extends RuntimeException {
    private final ErrorCode errorCode;
}
```

 * 전역으로 사용할 CustomException 입니다.
 * RuntimeException 을 상속받아서 Unchecked Exception 으로 활용합니다.
 * 생성자로 ErrorCode 를 받습니다.


### ErrorResponse

```
@Getter
@Builder
public class ErrorResponse {
    private final LocalDateTime timestamp = LocalDateTime.now();
    private final int status;
    private final String error;
    private final String code;
    private final String message;

    public static ResponseEntity<ErrorResponse> toResponseEntity(ErrorCode errorCode) {
        return ResponseEntity
                .status(errorCode.getHttpStatus())
                .body(ErrorResponse.builder()
                        .status(errorCode.getHttpStatus().value())
                        .error(errorCode.getHttpStatus().name())
                        .code(errorCode.name())
                        .message(errorCode.getDetail())
                        .build()
                );
    }
}
```

 * 실제로 유저에게 보낼 응답 Format 입니다.
 * 일부러 500 에러 났을 때랑 형식을 맞췄습니다. status, code 값은 사실 없어도 됩니다.
 * ErrorCode 를 받아서 ResponseEntity<ErrorResponse> 로 변환해줍니다.


### @ControllerAdvice and @ExceptionHandler

@ControllerAdvice 는 프로젝트 전역에서 발생하는 모든 예외를 잡아줍니다.

@ExceptionHandler가 하나의 클래스에 대한 것이라면, @ControllerAdvice는 모든 @Controller 즉, 전역에서 발생할 수 있는 예외를 잡아 처리해주는 annotation

ControllerAdvide에는 @ControllerAdvice와 @RestControllerAdvice가 존재

@ExceptionHandler 는 발생한 특정 예외를 잡아서 하나의 메소드에서 공통 처리해줄 수 있게 해줍니다.

따라서 둘을 같이 사용하면 모든 예외를 잡은 후에 Exception 종류별로 메소드를 공통 처리할 수 있습니다.

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value = { ConstraintViolationException.class, DataIntegrityViolationException.class})
    protected ResponseEntity<ErrorResponse> handleDataException() {
        log.error("handleDataException throw Exception : {}", DUPLICATE_RESOURCE);
        return ErrorResponse.toResponseEntity(DUPLICATE_RESOURCE);
    }

    @ExceptionHandler(value = { CustomException.class })
    protected ResponseEntity<ErrorResponse> handleCustomException(CustomException e) {
        log.error("handleCustomException throw CustomException : {}", e.getErrorCode());
        return ErrorResponse.toResponseEntity(e.getErrorCode());
    }
}
```


 * View 를 사용하지 않고 Rest API 로만 사용할 때 쓸 수 있는 @RestControllerAdvice 를 사용합니다.
 * handleDataException 메소드에서는 hibernate 관련 에러를 처리합니다.
 * handleCustomException 메소드는 직접 정의한 CustomException 을 사용합니다.
 * Exception 발생 시 넘겨받은 ErrorCode 를 사용해서 사용자에게 보여주는 에러 메세지를 정의합니다.


### @ControllerAdvice 범위설정

@ControllerAdvice는 모든 에러를 잡아주기 때문에 일부 에러만 처리하고 싶을 경우에는 따로 설정을 해주면 된다.

 * 1. 어노테이션
 * 2. basePackages (+basepackagesClasses)
 * 3. assignableTypes

예1)
```java
// 1.
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// 2.
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// 3.
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```
 * basePackages: 탐색 패키지 지정, org.example.controllers 패키지, 하위 패키지까지 모두 탐색
 * basePackagesClasses: 탐색 클래스 지정, 클래스의 맨 위에 있는 package부터 시작


예2)
```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```
=> 스프링 공식 문서 예제에서 보는 것 처럼 특정 애노테이션이 있는 컨트롤러를 지정할 수 있고, 특정 패키지를 직접 지정할 수도 있다. 패키지 지정의 경우 해당 패키지와 그 하위에 있는 컨트롤러가 대상이 된다. 그리고 특정 클래스를 지정할 수도 있다. 대상 컨트롤러 지정을 생략하면 모든 컨트롤러에 적용된다.


**주의사항**

어노테이션, 베이스패키지 등 설정자들은 runtime시 수행되기 때문에 너무 많은 설정자들을 사용하면 성능이 떨어질 수 있다!

>Use selectors such as annotations, basePackageClasses, and basePackages (or its alias value) to define a more narrow subset of targeted controllers.
>
>If multiple selectors are declared, boolean OR logic is applied, meaning selected controllers should match at least one selector. Note that selector checks are performed at runtime, so adding many selectors may negatively impact performance and add complexity.


### Exception 사용

```java
@RequiredArgsConstructor
@Service
public class MemberService {
    private final MemberRepository memberRepository;

    @Transactional
    public boolean follow(Long memberId) {
        Member currentMember = getCurrentMember();

        // 팔로우할 상대방 정보가 없는 경우
        Member targetMember = memberRepository.findById(memberId)
                .orElseThrow(() -> new CustomException(MEMBER_NOT_FOUND));

        // 자기 자신을 팔로우 하려는 경우
        if (currentMember.equals(targetMember))  {
            throw new CustomException(CANNOT_FOLLOW_MYSELF);
        }

                // code...
    }
}
```

 * 간단항 예제에서 상대방의 Member ID 를 입력 받아서 팔로우 하는 기능.
 * Exception 에 담겨지는 ErrorCode 만 보고도 어떤 종류의 문제가 발생한 건지 알 수 있게 되었다.
 * 또한 불필요하게 여러 Exception 을 만들지 않고 ErrorCode 만 새로 추가하면 사용 가능하다.


MEMBER_NOT_FOUND 실제 응답
```json
{
  "timestamp": "2021-03-14T03:29:01.878659",
  "status": 404,
  "error": "NOT_FOUND",
  "code": "MEMBER_NOT_FOUND",
  "message": "해당 유저 정보를 찾을 수 없습니다"
}
```


CANNOT_FOLLOW_MYSELF 실제 응답
```json
{
  "timestamp": "2021-03-14T03:16:25.98361",
  "status": 400,
  "error": "BAD_REQUEST",
  "code": "CANNOT_FOLLOW_MYSELF",
  "message": "자기 자신은 팔로우 할 수 없습니다"
}
```




---
# 실무에서의 예외처리

실무에선 정해져 있지는 않지만, 경험상 에러 인터페이스 정의를 제대로 해야한다.
무슨 얘기냐면 에러메시지로 나가는 포맷이 일정해야한다는 얘기다.
만약 로그인 모듈에서 발생한 예외에 응답하는 메세지는 에러코드랑 설명을 리턴해준다고 하고, 배송 모듈에서 발생한 예외는 에러코드랑 에러가 난 배송 번호를 리턴해준다고 하자.
그러면 @ControllerAdvice를 이용해서 통합으로 처리하려고 했지만 리턴 타입이 다르니까 통합해서 처리할 수 없다.
HTTP 상태코드, ErrorResponse같은 경우는 좋은 예다.
에러 인터페이스, 포맷이 다 같고 클라이언트 측에서도 이해하기 좋은 에러가 날라오는 것이다.

그래서 @ExceptionHandler와 함께 @ResponseStatus(value = HttpStatus.UNAUTHORIZED)
이런 것도 집어넣어서 일괄적으로 HTTP상태코드를 리턴하기도 한다.
다시 한 번 정리하지만 에러 메시지가 잘 정의되어있어야 하는게 전제 조건이다.

```java
public enum LoginErrorCode {
        OperationNotAuthorized(6000,"Operation not authorized"),
        DuplicateIdFound(6001,"Duplicate Id"),
        //...
        UnrecognizedRole(6010,"Unrecognized Role");
        private int code;
        private String description;
        private LoginErrorCode(int code, String description) {
            this.code = code;
            this.description = description;
        }
        public int getCode() {
            return code;
        }
        public String getDescription() {
            return description;
        }
    }
```

보통 에러를 위와 같이 한 곳에 정리를 할 것이다.
저렇게 미리 정의해놓고 실제 사용할 때는 LoginErrorCode.OperationNotAuthorized.getCode(); 이런식으로 불러와서 에러객체를 만들어서 리턴할 것이다.

그러면 1차적으로 에러객체 관리는 위와 같은 방법으로 끝난다.

그 다음에 아까 배운 @ControllerAdvice, @ExeptionHandler를 이용해서 에러를 처리해본다.
만약 @ControllerAdvice, @ExceptionHandler로 처리할 때, InvalidArgumentProvided 라는 에러코드를 만들었다면, 모든 컨트롤러에서 들어오는 인자(arguments)에 대해서 한 곳에서 처리하게 되므로, 중복된 코드를 쓰지 않게 된다.
그로인해 비즈니스 로직에 더 집중할 수 있고, 코드도 간단하게 조건문에 따라 throw new XXXXException(); 하고 호출해버리면 끝나기 때문에 유지보수에 아주 큰 도움이 된다



---
# HTML 화면 오류 vs API 오류
https://develop-writing.tistory.com/102



---
# Tip

### ExceptionHandler 우선순위

ExceptionResolver의 우선순위가 왜 ExceptionHandlerExceptionReosolver, ResponseStatusExceptionResolver, DefaultHandlerExceptionResolver의 순서로 되어잇는지 코드를 확인해봤습니다.
ExceptionResolver의 우선 순위를 확인할 수 있는 코드는 아래와 같습니다. WebMvcConfigurationSupport 클래스에 addDefaultHandlerExceptionResolvers 메소드는 아래처럼 정의되어 있습니다.

```java
protected final void addDefaultHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers,
      ContentNegotiationManager mvcContentNegotiationManager) {

   ExceptionHandlerExceptionResolver exceptionHandlerResolver = createExceptionHandlerExceptionResolver();
   exceptionHandlerResolver.setContentNegotiationManager(mvcContentNegotiationManager);
   exceptionHandlerResolver.setMessageConverters(getMessageConverters());
   exceptionHandlerResolver.setCustomArgumentResolvers(getArgumentResolvers());
   exceptionHandlerResolver.setCustomReturnValueHandlers(getReturnValueHandlers());
   if (jackson2Present) {
      exceptionHandlerResolver.setResponseBodyAdvice(
            Collections.singletonList(new JsonViewResponseBodyAdvice()));
   }
   if (this.applicationContext != null) {
      exceptionHandlerResolver.setApplicationContext(this.applicationContext);
   }
   exceptionHandlerResolver.afterPropertiesSet();
   exceptionResolvers.add(exceptionHandlerResolver);

   ResponseStatusExceptionResolver responseStatusResolver = new ResponseStatusExceptionResolver();
   responseStatusResolver.setMessageSource(this.applicationContext);
   exceptionResolvers.add(responseStatusResolver);

   exceptionResolvers.add(new DefaultHandlerExceptionResolver());
}
```
순서대로 List에 add를 하고,
나중에 DispatcherServlet의 processExceptionHandler 메소드에서 List를 순회하기 때문에 이 우선순위대로 동작한다
