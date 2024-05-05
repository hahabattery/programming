---
layout: default
title: Design Patterns In Spring
parent: Design Patterns
nav_order: 10
---

스프링 소스를 이해하기위해서 내부에서 이용되는 디자인패턴을 정리한다. 정리한 내용으로 보다 더 빨리 스프링 소스를 이해할 수 있게 된다.

---
# 스프링에서만 언급되는 패턴

"Design Patterns in the Spring" 키워드로 검색하면 관련 정보를 얻을 수 있다.


### 의존성 주입(Dependency Injection)

객체 자체가 아니라 Framework에 의해 객체의 의존성이 주입되는 설계 패턴으로 Framework에 의해 동적으로 주입되므로 여러 객체 간의 결합을 줄일 수 있습니다.

Spring IoC 컨테이너에 빈을 어떻게 등록되는가?

1. Component Scanning<br>
  @ComponentScan 어노테이션과 @Component 어노테이션을 사용하여 빈을 등록하는 방법<br>
  @ComponentScan :  어느 지점부터 컴포넌트를 찾는 지 정의(지정한 지점 패키지부터 하위 패키지에 모든 @Component 어노테이션이 붙은 클래스를 찾음)<br>
  @Component : 빈으로 등록할 클래스<br>
  @Controller, @Service, @Repository, @Configuration에는 @Component가 정의되어 있습니다.

2. 빈 설정파일에 직접 빈을 등록<br>
  빈 설정파일은 XML과 자바파일로 작성할 수 있습니다.

**XML 파일로 작성할 경우**

1. <bean> 태그를 이용하여 bean을 설정합니다.
2. bean에 대한 설정은 <property> 태그를 이용하여 설정합니다.(메소드명)
3. <ref> 태그로 setter에 주입할 bean(객체)의 이름을 설정합니다.

```xml
<bean id="bean의 이름" class="패키지.클래스"/>
<bean id="bean의 이름" class="패키지.클래스">
	<property name="set을 뺀 메소드명">
		<ref bean="setter에 주입할 bean의 름"/>
 혹은

		<bean class="패키지명.클래스"/>
	</property>
</bean>
<bean id="bean의 이름" class="패키지.클래스">
	<property name="set을 뺀 메소드명" ref="bean의 이름"/>
</bean>
```

**자바 파일로 작성할 경우**

1. 설정용 클래스를 만들고 클래스 위에 @Configuration 어노테이션을 추가합니다.
2. @Bean 어노테이션을 사용하여 직접 빈을 정의합니다.

name이 없는 경우 메소드명이 bean의 이름으로 정의됩니다.

```java
@Configuration
public class Configuration {

  @Bean(name = "bean의 이름")
  public TestBean testBean {
    return new TestBean; // bean에 등록할 클래스
  } //testBean

} //class
```

**등록된 빈을 어떻게 얻는가?**

1. 어노테이션 이용(@Resource, @AutoWired)
2. 클래스 이용(AnnotationConfigApplicationContext)

```java
public class UseBean {
  // 1번
  // 어노테이션 이용(@Resource, @AutoWired)
  @Resource(name = "bean의 이름")
  private TestBean testBean;
  // 2번
  // AnnotationConfigApplicationContext 객체 이용
  private TestBean testBean;
  public void useBean() {
    // 해당 패키지에서 bean 스캔
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("패키지");
    // bean에 등록된 객체 얻기
    testBean = (TestBean) context.getBean("bean의 이름");
  } //useBean
} //class
```

**의존성 주입(DI)의 장점**

1. 종속성을 줄일 수 있다.
2. 재사용성이 증가합니다. 일부 인터페이스의 다른 구현이 필요한 경우, 코드를 변경할 필요없이 해당 구현을 사용하도록 components를 구성할 수 있다.


### 프론트 컨트롤러(Front Controller)
"Front Controller" 키워드로 검색하면 관련 정보를 찾을 수 있다.

### 뷰 헬퍼(View Helper)

View에서 제어 로직, 데이터 액세스, 화면 구성 등을 모두 넣는 것이 아닌, View는 단순히 클라이언트에게 정보를 보여주는 역할, Controller는 View와 Model을 연결해주는 역할, helper는 View에서 사용할 데이터 모델을 적용하는 것을 포함하여 데이터를 가공하는 역할을 합니다. Spring에서는 보통 View(jsp), Controller(controller), Service(helper), DAO(helper)로 업무 로직과 데이터 액세스 로직을 나누어 Service, DAO 클래스를 헬퍼로 지정하는 방식을 사용합니다.


---
# ProxyPattern
Spring에서 Proxy Pattern을 이용하는 방법은 다음과 같다.

* AspectJ
  + https://github.com/bitedd/Examples/blob/master/Springs/aop-Aspectj.md
* JDKProxy-CGLib
  + https://github.com/bitedd/Examples/blob/master/Springs/aop-JDKProxy-CGLib.md

---
# Singleton Pattern
스프링에서는 클래스 자체에 의해서 객체를 생성하지 않고, 스프링 컨테이너에 의해 구현된다. 스프링에서는 컨테이너 내에서 특정 클래스에 대해 @Bean이 정의되면, 스프링 컨테이너는 그 클래스에 대해 하나의 인스턴스를 만든다. 이 공유 인스턴스는 설정 정보에 의해 관리되고, bean이 호출될 때마다 스프링은 생성된 공유 인스턴스를 반환한다. 이미 생성된 공유 인스턴스를 반환하기 때문에 싱글턴 패턴과 동일합니다. 이와 같은 개념은 의존성 주입과도 연관되어 있다.


---
# Template Method Pattern
슈퍼 클래스에서 정의한 기본 로직을 서브 클래스에서 필요에 맞게 구현해서 쓰는 방법

![](/images/design-patterns/template-method-pattern-240114.png)

### 예 1
스프링에서 웹과 관련된 처리를 하기 위해서  WebMvcConfigurer interface를 구현하여 사용할 수 있다. 해당 인터페이스에는 Spring MVC에서 아주 유용하게 사용되는 기능들이 선언되어 있어서 해당 기능을 사용하려면 관련 메서드를 재정의 하면된다. WebMvcConfigurer의 모든 메서드가 default 메서드로 선언되어 있어 반드시 재정의할 필요가 없고, 필요할 때 재정의하여 사용할 수 있다. 스프링에서 요청을 받아 처리하려면 Controller가 필요한데 addViewController를 구현하여 클래스 없이 특정 요청을 바로 View Resolver에게 View명을 넘길 수 있다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    // 시작페이지 url을 '/'가 아닌 home으로
    registry.addRedirectViewController("/", "home");
    // registry.addViewController("/").setViewName("home");
  } //addViewControllers

} //class
```



---
# Factory Method Pattern
오브젝트 생성 방법을 기본 코드에서 독립시키는 방법

![](/images/design-patterns/fatory-method-pattern-240114.png)

### 예 1
예시) Spring 어노테이션 및 Java를 이용한 언어 Factory 클래스 생성하기

해당 예제는 팩토리 패턴과 전략 패턴 모두 이용한 예제로 볼 수 있습니다.

1. 요구사항을 담은 interface 생성

```java
public interface Language {

    public void compile();

    public String getLanguageType();
}
```


2. 상속받은 클래스를 구분해 줄 클래스 생성

```java
public class LanguageType {

    public static final String JAVA = "java";

    public static final String PYTHON = "python";

}
```

3. interface를 상속받아 구현할 클래스 생성

```java
public class Java implements Language{

    @Override
    public void compile() {
        System.out.println("Java Compile");
    }

    @Override
    public String getLanguageType() {
        return LanguageType.JAVA;
    }
}

public class Python implements Language{

    @Override
    public void compile() {
        System.out.println("Python Compile");
    }

    @Override
    public String getLanguageType() {
        return LanguageType.PYTHON;
    }
}
```


4. 파라미터에 따라 객체를 반환해줄 Factory 클래스 생성

```java
// 기본 Factory 클래스
@Component
public class LanguageFactory {

    public Language getLanguage(String languageType) {

        if(languageType.equal(LanguageType.JAVA)){
            return new Java();
        } else if(languageType.equal(LanguageType.Python)) {
            return new Python();
        } else {
            return null;
        }//end else

    }//getLanguage

}//class
```

```java
// 개선된 Factory 클래스
@Component
public class LanguageFactory {

    @Autowired
    private List<Language> languageList;

    public Language getLanguage(String languageType) {
        for(Language language : languageList) {
            if(languageType.equals(language.getLanguageType()))
                return language;
        }//end for
        return null;
    }//getLanguage

}//class
```

5. Factory 클래스를 사용하여 객체를 받아 사용하는 클래스 생성

```java
@Service
public class UseFactory {

    @Autowired
    private LanguageFactory factory;

    @Override
    public void useLanguageFactory() {
        Language java = factory.getLanguage(LanguageType.JAVA);
        Language python = factory.getLanguage(LanguageType.PYTHON);

        java.compile();
        System.out.println(java.getClass());

        python.compile();
        System.out.println(python.getClass());

    }//useLanguageFactory

}//class
```

---
# ApplicationEventPublisher

ApplicationEventPublisher 은 옵저버 패턴의 구현체로 이벤트 프로그래밍에 필요한 기능을 제공한다.

```java
public class Member extends ApplicationEvent {

    String name;

    public Member(Object source, String name) {
        super(source);
        this.name = name;
    }
    //getter, setter 생략
}
```

Event를 처리할 Listener를 만든다. (해당 Listener 는 Bean으로 등록 되어 있어야 함.)

```java
@Component
public class SampleListener implements ApplicationListener<Member> {

    @Override
    public void onApplicationEvent(Member member) {
        System.out.println("반가워요 "+member.getName());
    }
}
```

마지막으로 이벤트를 발생 시켜 주면 된다.

```java
@Component
public class ApplicationRunner implements ApplicationRunner {

    @Autowired
    ApplicationEventPublisher applicationEventPublisher;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Member member = new Member(this,"김재준");
        applicationEventPublisher.publishEvent(member);
    }
}
```
이렇게 Member 에 대한 Event를 생성 후 event를 발생 시킬 수 있다.

하지만 Member.Class 가 ApplicationEvent 를 상속 받음으로써 Spring 와 강한 의존성이 생겼다.

이러한 문제점을 해결 하기 위해 Spring 4.3 이후로 부터는 POJO 스럽게 Event 를 등록 할 수 있다.

먼저 Member.Class 에 있는 상속 구조부터 제거하자

```java
public class Member{
    String name;

    public Member(String name) {
        this.name = name;
    }
    //getter, setter 생략
}
```
또한 Listener 도 ApplicationListener 대신 @EventListener 를 이용하여 대체가 가능하다.

```java
@Component
public class SampleListener  {

    @EventListener
    public void MemberCreateEvent(Member member) {
        System.out.println("반가워요 "+member.getName());
    }
}
```
이제 똑같이 실행 해주면 아까의 코드와 동일하게 작동한다.

```java
@Component
public class ApplicationRunner implements org.springframework.boot.ApplicationRunner {

    @Autowired
    ApplicationEventPublisher applicationEventPublisher;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Member member = new Member("김재준");
        applicationEventPublisher.publishEvent(member);
    }
}
```
앞서 말한 Event 등록은 ApplicationContext 가 생성되고 나서 작동하는 Event 들 이다.

ApllicationContext 가 등록되기 이전의 Event 를 발생 하고 싶으면 다음과 같은 설정을 해주면 된다.

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(DemoApplication.class);
        springApplication.addListeners(new SampleListener()); // Listeners를 등록 한다.
        springApplication.run(args);
    }
}
```
.addListeners() 이 ApplicationListener<?> 타입을 파라미터를 받기 때문에 앞서 말한 @EventListener 를 사용하지 못한다.

```java
@Component
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("ApplicationContext 가 생성하기전에 출력해 ");
    }
}
```
또한 ApplicationListener interface 는 FunctionalInterface 임으로 람다식으로 간단히 표현 할 수있다.

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(DemoApplication.class);
        springApplication.addListeners((ApplicationListener<ApplicationStartingEvent>) applicationEvent -> {
            System.out.println("이렇게 할수도 있어요");
        });
        springApplication.run(args);
    }
}
```

