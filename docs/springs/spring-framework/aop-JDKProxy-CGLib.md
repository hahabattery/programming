---
layout: default
title: AOP JDKProxy CGLib
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---

Spring에서 사용하는 AOP관련 2가지 구현체는 JDK Proxy(Dynamic Proxy)와 CGLib가 있다.

# 리소스
 * https://velog.io/@ann0905/AOP%EC%99%80-Transactional%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC  <= 튜토리얼
 * https://velog.io/@suhongkim98/JDK-Dynamic-Proxy%EC%99%80-CGLib
   + => 이것도 Spring AOP가 헷갈릴 떄 보면 좋을 듯. JDK Proxy와 Spring AOP에 대한 예제도 포함되있어서 차분히 읽으면 좋을 듯.

---
# AOP( Aspect Oriented Programming; 관점 지향 프로그래밍)에 대한 기본적인 내용

객체 지항 프로그래밍(OOP)에서는 주요 관심사에 따라 클래스를 분할한다. 이 클래스들은 보통 SRP(Single Responsibility Principle)에 따라 하나의 책임만을 갖게 설계된다.
하지만 클래스를 설계하다보면 로깅, 보안, 트랜잭션 등 여러 클래스에서 공통적으로 사용하는 부가 기능들이 생긴다. 이들은 주요 비즈니스 로직은 아니지만, 반복적으로 여러 곳에서 쓰이는 데 이를 흩어진 관심사(Cross Cutting Concerns)라고 한다.

따라서 흩어진 관심사를 별도의 클래스로 모듈화하여 위의 문제들을 해결하고, 결과적으로 OOP를 더욱 잘 지킬 수 있도록 도움을 주는 것이 AOP이다.

### AOP의 주요 개념

 * Aspect: Advice + PointCut로 AOP의 기본 모듈
 * Advice: Target에 제공할 부가 기능을 담고 있는 모듈
 * Target: Advice이 부가 기능을 제공할 대상 (Advice가 적용될 비즈니스 로직)
 * JointPoint: Advice가 적용될 위치
   + 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용 가능
 * PointCut: Target을 지정하는 정규 표현식

---
# Proxy 구현체
Spring AOP는 Proxy 디자인 패턴을 사용하여 구현된다.

Spring에서 사용하는 2가지 구현체는 JDK Proxy(Dynamic Proxy)와 CGLib가 있다.

JDK Proxy는 Reflection을 이용하고, CGLib는 Byte는 Code Instrument룰 이용한다.

![](./doc/imgs/jdk-cglib-proxy-01.png)



---
# Spring과 Spring Boot
 * Spring은 기본적으로 인터페이스를 구현한 객체이면 JDK Proxy, 클래스로 구현된 객체이면 CGLib 로 동작한다.

 * Spring Boot에서는 기본적으로 CGLib 방식을 이용하는데, JDK Proxy 방식으로 변경하기 위해서는
 * spring.aop.proxy-target-class 설정을 false로 하면 JDK Proxy로 동작한다.
 * 관련 글
   + https://private-space.tistory.com/98

 * Dynamic Proxy와 CGLib은 모두 런타임 위빙 방식이며 프록시 패턴으로 동작한다. 따라서 메서드 실행 시에만 위빙이 가능하다. 그래서 Dynamic Proxy와 CGLib를 사용하는 스프링 AOP도 메서드 실행 조인포인트만 지원한다.

---
# JDK Proxy 개요

JDK Proxy의 경우 AOP를 적용하여 구현된 클래스의 인터페이스를 프록시 객체로 구현해서 코드를 끼워넣는 방식이다.

구체적인 구현 코드는 아래와 같다.
```java
public class BoardServiceProxy {
    private final BoardService boardService;
    private final TransactonManager manager = TransactionManager.getInstance();

    public BaordServiceProxy (BoardService boardService) {
        this.boardService = boardService;
    }

    public void writePost(PostDTO postDTO) {
        try {
            manager.begin();
            boardService.writePost(postDTO);
            manager.commit();
        } catch ( Exception e ) {
            manager.rollback();
        }
    }
}
```

Springboot의 경우 기본적으로 프록시 객체를 생성할 때 CGLib를 사용하고 있다.

그 이유는, JDK Proxy가 프록시 객체를 생성할 때 내부적으로 Reflection을 사용하고 있기 때문이다.

리플렉션 자체가 비용이 비싼 API이기도 하고 Effective Java 책에서도 리플렉션을 가급적 사용하지 않는 것을 추천하고 있다.

게다가 JDK Proxy의 경우 AOP 적용을 위해서 반드시 인터페이스를 구현해야한다는 단점이 있다.

우리가 의무적으로 서비스 계층에서 인터페이스 -> XXXXImpl 클래스를 작성하던 관례도 다 이러한 JDK Proxy의 특성 때문이기도 하다. (물론 이 것이 항상 나쁜것은 아니다. 장점도 분명 있다. )

```java
public class ExamDynamicHandler implements InvocationHandler {
    private ExamInterface target; // 타깃 객체에 대한 클래스를 직접 참조하는것이 아닌 Interface를 이용

    public ExamDynamicHandler(ExamInterface target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        // TODO Auto-generated method stub
        // 메소드에 대한 명세, 파라미터등을 가져오는 과정에서 Reflection 사용
        String ret = (String)method.invoke(target, args); //타입 Safe하지 않는 단점이 있다.
        return ret.toUpperCase(); //메소드 기능에 대한 확장
    }
}
```
Dynamic Proxy는 InvocationHandler라는 인터페이스를 구현한다.

InvocationHandler의 invoke 메소드를 오버라이딩 하여 Proxy 위임 기능을 수행하는데, 이 때 메소드에 대한 명세와 파라미터를 가져오는 과정에서 리플렉션을 사용한다.

JDK Proxy의 경우 자바에서 기본적으로 제공하고 있는 기능이다.


---
# CGLib 개요

반면에 CGLib의 경우 외부 3rd party Library이며 JDK Proxy와는 달리 리플렉션을 사용하지 않고 바이트코드 조작을 통해 프록시 객체 생성을 하고 있다.

게다가 인터페이스를 구현하지않고도 해당 구현체를 상속받는 것으로 문제를 해결하기 떄문에 성능상에 더욱 이점이 있다.

CGLib는 Enhancer라는 클래스를 바탕으로 Proxy를 생성한다.
```java
// 1. Enhancer 객체를 생성
Enhancer enhancer = new Enhancer();
// 2. setSuperclass() 메소드에 프록시할 클래스 지정
enhancer.setSuperclass(BoardServiceImpl.class);
enhancer.setCallback(NoOp.INSTANCE);
// 3. enhancer.create()로 프록시 생성
Object obj = enhancer.create();
// 4. 프록시를 통해서 간접 접근
BoardServiceImpl boardService = (BoardServiceImpl)obj;
boardService.writePost(postDTO);
```


이처럼 상속을 통해 프록시 객체가 생성되기 때문에 더욱 성능상에 이점을 누릴 수 있다.
```java
BoardServiceProxy.writePost(postDTO) -> BoardServiceImpl.writePost(postDTO)
```

enhancer.setCallback(NoOp.INSTANCE);라는 코드가 존재하는데 이는 Enhancer 프록시 객체가 직접 원본 객체에 접근하기 위한 옵션이다.

기본적으로 프록시 객체들은 직접 원본 객체를 호출하기 보다는, 별도의 작업을 수행하는데 CGLib의 경우 Callback을 사용한다.

CGLib에서 가장 많이 사용하는 콜백은 net.sf.cglib.proxy.MethodInterceptor인데, 프록시와 원본 객체 사이에 인터셉터를 두어 메소드 호출을 조작하는 것을 도와줄 수 있게 된다.

```java
BoardServiceProxy -> BoardServiceInterceptor -> BoardServiceImpl
```

그리하여
자바 리플렉션 방식보다 CGLib의 MethodProxy이 더 빠르고 예외를 발생시키지 않는다고 하여 Springboot에서는 CGLib를 기본 프록시 객체 생성 라이브러리로 채택하게 되었다.


---
# 제약사항
AOP를 이용하는 경우 아래와 같은 제한 사항이 생기게 된다.

 * 같은 객체내의 method끼리 호출시 AOP가 동작하지 않습니다.
 * RTW(Runtime Weaving)으로 처리 되기 때문에 약간의 성능저하가 있습니다.


# @Transactional

1. private은 트랜잭션 처리를 할 수 없다.
앞서 트랜잭션이 코드 레벨에서 어떻게 동작하는지 대락적으로 살펴봤다. 프록시 객체는 타겟 객체/인터페이스를 상속 받아서 구현하는데, private으로 되어 있으면 자식인 프록시 객체에서 호출할 수 없다.

2. 트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다.
아래 예와 같은 경우 B() 메소드를 호출하는 경우에는 내부에서 트랜잭션이 걸려있는 다른 메서드를 호출한다 할지라도 적용되지 않는다.
```java
@Service
public class TestService {

	@Autowired
	CouponGroupMapper couponGroupMapper;

	@Transactional
	public void A(CouponGroupParam param) {
		param.setStatus(CouponGroupStatus.CREATED);	// 상태 변경
		couponGroupMapper.insertCouponGroup(param);
	}

	public void B() {
		for(int i=0; i<3; i++) {
			CouponGroupParam param = new CouponGroupParam();
    		param.setName("1000포인트 쿠폰");
    		param.setAmount(1000);
    		param.setMaxCount(100);
    		param.setValidFromDt(new Date());
    		param.setValidToDt(new Date());
    		param.setIssuerId("0101");
    		param.setCode("B000" + i);

			A(param);
		}

		throw new RuntimeException(); // 오류 발생!
	}

	@Transactional
	public void C() {
		for(int i=0; i<3; i++) {
			CouponGroupParam param = new CouponGroupParam();
    		param.setName("1000 포인트 쿠폰");
    		param.setAmount(1000);
    		param.setMaxCount(100);
    		param.setValidFromDt(new Date());
    		param.setValidToDt(new Date());
    		param.setIssuerId("0101");
    		param.setCode("C000" + i);

			A(param);
		}

		throw new RuntimeException(); // 오류 발생!
	}
}
```


---
# 정리

### @Transactional 도 Spring AOP 를 이용해서 프록시 방식으로 동작한다.
 * https://cobbybb.tistory.com/17#%EC%A-%--%EC%-E%--%EC%A-%--%EC%--%--%--Transaction%EC%-D%B-%--%EC%--%--%EA%B-%A-%--%EC%--%--%EC%--%--%EC%--%-C%--%ED%--%B-%EC%B-%-C%EB%--%--%EB%-A%--%--method%EC%--%--%EB%A-%-C%--Transaction%EC%-D%B-%--%EC%-E%--%EB%-B%A-%EB%A-%B---%-F
   + 여러가지 실험으로 이해하기 쉽게 설명되어있음. (springs/spring-boot/transaction-proxy 폴더의 예제 소스 참조)

 * @Transactional은 Proxy 패턴(AOP를 이용)으로 동작한다.

### private은 @Transactional이 적용되지 않는다.
 * 원본 클래스/인터페이스를 상속 받아 프록시를 생성하기 때문에 접근 제어가 private으로 되어 있으면 안된다.



### 같은 클래스 내에서의 여러 @Transactional 호출

##### 진입점에 Transactional이 있고, 안에서 Transactional이 걸려있는 메소드 호출시

 * Proxy 형태로 동작하기 때문에 같은 Class안의 method에서 다른 method를 호출했을때 중첩 Transaction이 동작하지 않음 —> 무조건 진입점의 Transaction기준으로 동작함

##### 진입점에 Transaction이 없고 안에서 호출되는 method에만 Transaction이 있다면

 * 객체 외부에서 처음으로 진입하는 메서드에 트랜잭션 처리가 되어 있어야, 해당 요청을 프록시 객체가 대신 처리할 수 있다. 따라서 트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다.



### 당연하겠지만, @Transactional 애너테이션은 빈(bean)인 객체에만 적용되니 주의해야 합니다. 일반 객체의 메소드에 작성하여도 정상적으로 동작하지 않습니다.
