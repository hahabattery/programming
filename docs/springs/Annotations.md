---
layout: default
title: Annotations
parent: Springs
nav_order: 4
---

# Annotations
{: .no_toc}

# Table of contents
{: .no_toc .text-delta }

1. TOC 
{:toc}



어노테이션을 기억하기 위해서 정리하는 공간




### @Value

[Spring @Value Annotation](https://www.digitalocean.com/community/tutorials/spring-value-annotation)



---
# Spring MVC

### @Configuration

 * org.springframework.context.annotation의 Configuration 애노테이션과 Bean 애노테이션 코드를 이용하여 스프링 컨테이너에 새 로운 빈 객체를 제공할 수 있다
 

### @EnableWebMvc 에 대해서

 * DispatcherServlet의 RequestMappingHandlerMapping, RequestMappingHandlerAdapter, ExceptionHandlerExceptionResolver, MessageConverter 등 Web에 필요한 빈들을 대부분 자동으로 설정해준다.
 * xml로 설정의 <mvc:annotation-driven/> 와 동일하다.
 * 기본 설정 이외의 설정이 필요하다면 WebMvcConfigurerAdapter 를 상속받도록 Java config class를 작성한 후, 필요한 메소드를 오버라이딩 하도록 한다.

### @RequestMapping

 * Http 요청과 이를 다루기 위한 Controller 의 메소드를 연결하는 어노테이션
 * Http Method 와 연결하는 방법
   + @RequestMapping(value="/users", method=RequestMethod.POST)
   + From Spring 4.3 version (@GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping)
 * Http 특정 해더와 연결하는 방법
   + @RequestMapping(method = RequestMethod.GET, headers = "content-type=application/json")
 * Http Parameter 와 연결하는 방법
   + @RequestMapping(method = RequestMethod.GET, params = "type=raw")
 * Content-Type Header 와 연결하는 방법
   + @RequestMapping(method = RequestMethod.GET, consumes = "application/json")
 * Accept Header 와 연결하는 방법
   + @RequestMapping(method = RequestMethod.GET, produces = "application/json")


---
# 메소드

### Spring MVC가 지원하는 메소드 인수 애노테이션

 * @RequestParam
 * @RequestHeader
 * @RequestBody
 * @RequestPart
 * @ModelAttribute
 * @PathVariable
 * @CookieValue
 

### @RequestParam

 * Mapping된 메소드의 Argument에 붙일 수 있는 어노테이션
 * @RequestParam의 name에는 http parameter의 name과 멥핑
 * @RequestParam의 required는 필수인지 아닌지 판단
 

### @PathVariable

 * @RequestMapping의 path에 변수명을 입력받기 위한 place holder가 필요함
 * place holder의 이름과 PathVariable의 name 값과 같으면 mapping 됨
 * required 속성은 default true 임
 

### @RequestHeader

 * 요청 정보의 헤더 정보를 읽어들 일 때 사용
 * @RequestHeader(name="헤더명") String 변수명
 

---
# Injection


### @Autowire



 * 1. 클래스의 property(field)에 @Autowire

```java
@Component("fooFormatter")
public class FooFormatter {
    public String format() {
        return "foo";
    }
}

@Component
public class FooService {  
    @Autowired
    private FooFormatter fooFormatter;
}
```
Spring injects fooFormatter when FooService is created.


 * 2. setter 메소드에 @Autowire

```java
public class FooService {
    private FooFormatter fooFormatter;
    @Autowired
    public void setFormatter(FooFormatter fooFormatter) {
        this.fooFormatter = fooFormatter;
    }
}
```
the setter method is called with the instance of FooFormatter when FooService is created.


 * 3. 생성자에 @Autowire

```java
public class FooService {
    private FooFormatter fooFormatter;
    @Autowired
    public FooService(FooFormatter fooFormatter) {
        this.fooFormatter = fooFormatter;
    }
}
```
an instance of FooFormatter is injected by Spring as an argument to the FooService constructor


 * Configuration 클래스내의 메소드에 @Autowire
any method annotated with @Autowired is a config method. It is called on bean instantiation after field injection is done. 
The arguments of the method are injected into the method on calling.

```java
// https://stackoverflow.com/questions/32381843/autowired-on-method-in-spring
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog mC,
            CustomerPreferenceDao cPD) {
        this.movieCatalog = mC;
        this.customerPreferenceDao = cPD;
    }
	 // ...

}
```
it's just an alternative to annotate the method with @PostConstruct; 
methods annotated with these annotations are called after bean creation and can be used to initialize the bean. @PostConstruct comes from JavaEE, whereas @Autowired is of Spring origin.


 * @Autowired and Optional Dependencies

```
// Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: 
// No qualifying bean of type [com.autowire.sample.FooDAO] found for dependency: 
// expected at least 1 bean which qualifies as autowire candidate for this dependency. 

public class FooService {
    @Autowired(required = false)
    private FooDAO dataAccessor; 
}
```
꼭 필요하지 않는 경우에는(이런 경우가 있긴한가?) "required" 를 false로 설정할 수 있다.


### @Autowired vs @Inject vs @Resource

|          |  @Autowired   | @Inject | @Resource |
|----------|:-------------:|:------:|------:|
|제공자 | Spring | Java | Java |
|연결방식| byType 먼저, 2개이상 있으면 byName(필드명, 파라미터명으로 빈이름을 매칭한다) | Type | byName 먼저, 못 찾으면 byType |
|byName 강제하기| @Qualifier("tire1") | | @Resource(name="tire1") |


> Spring uses the bean's name as a default qualifier value.


@Qualifier 같은 경우는 
1. @Qualifier끼리 매칭
2. 빈 이름 매칭
3. NoSuchBeanDefinitionException 예외 발생

@Primary는 우선순위를 정하는 방법이다. @Autowired시에 여러 빈이 매칭되면 @Primary가 우선권을 가진다.

@Primary는 기본값처럼 동작하는 것이고, @Qualifier는 매우 상세하게 동작한다. 이런 경우 스프링은 자동보다는 수동이 넓은 범위의 선택권보다는 좁은 범위의 선택권이 우선순위가 높다. 따라서 @Qualifier가 우선권이 높다.


### https://withseungryu.tistory.com/65 내용 정리

```
public class Animal{}
@Component
public class Dog implements Animal{}

@Component
public class Cat implements Animal{}
```

차이

```
@Autowired
private Dog cat 
//타입 기준 -- Dog 타입으로 연결

@Resource
private Dog cat
//이름 기준 -- Cat 타입으로 연결

@Inject
private Cat dog
//타입 기준 -- Cat 타입으로 연결
```

##### @Autowired의 특징

 * Spring Framwork에만 존재하기 때문에 타 프레임워크에서 사용 불가능하다.
 * 타입으로 먼저 검색하고, 그다음에 이름을 통해 빈을 검색한다.
 * @Autowired는 기본적으로 특정 빈을 찾지 못하면 예외를 던진다.
 * 이 때 @Autowired의 required 속성값을 false로 지정해 빈을 못 찾더라도 그냥 치나치게 해줄 수 있다.
 * 만약 타입을 기준으로 IoC 컨테이너에 호환 타입이 여럿 존재하거나 그룹형이 아닐 때
   + @Primary, @Qualifier를 통해 특정한 빈에 우선권을 부여 할 수 있다.

##### @Resource의 특징
 * 자바 측에서 @Autowired를 참고 해 만든 어노테이션.
 * **javax.annotation**에 속해 있음.
 * 이름을 통한 검색 방식이기 때문에, POJO가 여럿일 때 대상이 모호하지 않고, 명확하다(고 생각할 수도 있다).
 * @Autowired + @Qualifier


##### @Inject의 특징
 * 자바 측에서 @Autowired를 참고 해 만든 어노테이션
 * **javax.annotation**에 속해 있음.
 * 타입을 통한 검색방식
 * 타입이 같은 POJO가 여럿일 때는 아래처럼 커스텀 어노테이션(custom annotation)을 작성해야한다.

```
//커스텀 어노테이션
@Qualifier
@Target({ElementType.TYPE, ElementType.FILED, ElementType.PARAMETER})
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface ExampleAnnotation {
}
//
//커스텀 어노테이션을 작성한 후 빈 인스턴스를 생성하는 POJO 주입 클래스에 붙인다.
@ExampleAnnotation
public class Example{ }
//
//주입 지점에 커스텀 어노테이션을 함께 붙이면 모호해지지 않는다.
public calss ForExample{
	
    @Inject @ExampleAnnotation
    ...
  
}
```

※ 위 @Qualifier는 @Autowired의 @Qualifier와는 전혀 다르다.


###### 정리

 * @Autowired는 스프링 프레임워크에서 나왔기 때문에, 다른 프레임워크로 변환 할 때 호환이 안된다.
   따라서 타 프레임워크로도 호환을 원한다면 @Resource, @Inject를 사용하자.
   
 * 이름을 통해 검색 방식을 사용하고 싶으면 @Resource를 사용하고,
   타입을 통해 검색 방식을 사용하고 싶으면 @Autowired, @Inject를 사용하자.

 * @Configuration 으로 빈 생성시에는 함수이름이 빈 이름으로 사용되게 된다.
 * @RequiredArgConstructor를 이용하는 경우는 private final 에 @Qualifier("빈명")를 이용한 경우에 적용되지 않고, 변수명을 맞춰야지 매칭이 되었다.

![](doc/imgs/AutowirevsInjectvsResource.png)


# Lombok

* [@Data](https://projectlombok.org/features/Data)

