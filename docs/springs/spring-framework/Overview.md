---
layout: default
title: Spring Framework Overview
grand_parent: Springs
parent: Spring Framework
nav_order: 1
---

 * Spring MVC 를 사용하지만, Springboot를 이용하지 않는 프로젝트들의 소스들이다.

---
# Spring 다시 공부할 때 순서
최근에 spring boot로만 작업을 해서 많이 까먹은거 같다.
다시 spring boot가 아닌 프로젝트를 이용시에는 아래 순서로 하면 낫지 않을까...


* https://www.boostcourse.org/web316/lecture/16762?isDesc=false
  + boostcourse의 spring framework 강좌, 이거를 다 살펴봐야 한다. 한 3일정도 소요될 듯.

* [더블에스 Devlog Spring MVC 카테고리](https://walbatrossw.github.io/categories/spring-mvc)


* JPetStore 6 <= Spring MVC분석시에 도움이 될 듯하다.
https://github.com/mybatis/jpetstore-6

http://mybatis.org/jpetstore-6/ko/

> JPetStore 6 is a full web application built on top of MyBatis 3, Spring 5 and Stripes.

최대한 단순한 구조로 작성된 Spring MVC + mybatis 샘플

오래된 프로젝트여서 업데이트가 뜸한데, 그래도 최근에 업데이트되기도 함.


 * edwith의 부스트코스
   * https://www.boostcourse.org/web316/lecture/16762?isDesc=false
   * => Spring Framework 에 대해서 다시 학습이 필요할 때 보면 좋은 자료

 * Overview of Spring MVC Architecture
   * https://terasolunaorg.github.io/guideline/1.0.1.RELEASE/en/Overview/SpringMVCOverview.html

 * Spring MVC - DispatcherServlet 동작 원리
   * http://jess-m.tistory.com/15 [Jess's Home]

 * Spring MVC – Full java based config
   * https://samerabdelkafi.wordpress.com/2014/08/03/spring-mvc-full-java-based-config/

 * web.xml vs Initializer with Spring
   * https://www.baeldung.com/spring-xml-vs-java-config

 * Spring – Mixing XML and JavaConfig
   * https://mkyong.com/spring/spring-mixing-xml-and-javaconfig/


 * arey/spring-javaconfig-sample
   * https://github.com/arey/spring-javaconfig-sample/blob/master/src/main/webapp/WEB-INF/web.xml

 * WebMvcConfigurationSupport
   * https://docs.spring.io/spring-framework/docs/3.0.6.RELEASE_to_3.1.0.BUILD-SNAPSHOT/3.1.0.BUILD-SNAPSHOT/org/springframework/web/servlet/config/annotation/WebMvcConfigurationSupport.html


 * Spring mvc (2) 그리고 Spring boot
   * https://docs.spring.io/spring-framework/docs/3.0.6.RELEASE_to_3.1.0.BUILD-SNAPSHOT/3.1.0.BUILD-SNAPSHOT/org/springframework/web/servlet/config/annotation/WebMvcConfigurationSupport.html

 * Migrate Spring MVC servlet.xml to Java Config
   * https://www.luckyryan.com/2013/02/07/migrate-spring-mvc-servlet-xml-to-java-config/

 * Spring RequestMapping
   * https://www.baeldung.com/spring-requestmapping

---
# Spring MVC 소스 분석할 때 팁
* web.xml 설정내용 확인

---
# Maven Profile VS Spring Profile
빌드 및 배포를 할때 버전관리, 혹은 실제 DB 혹은 test용 DB 등 여러가지 환경을 바꾸어야 할 때가 있습니다.

이때 maven 의 profile 기능을 사용하게 되면, 특정 Build 환경에 맞춘 리소스, 환경 설정 등을 클릭 한번으로 해결할 수 있습니다.

maven profile 은 환경마다 빌드를 새로 해야 하는 문제가 있으니 이것보다는 spring profile 을 권장합니다.



---
# Spring MVC 프로젝트 생성하기
**웹에서 intellij spring mvc mybatis 형태로 검색하면 프로젝트를 생성하는 가이드를 찾을 수 있다**

STS 같은 경우는 Spring MVC용 프로젝트 생성을 IDE에서 지원해준다.

IntelliJ생성시는 아래 내용을 참고해서 작업하면 된다.

https://walbatrossw.github.io/spring-mvc/2017/11/22/intellij-springmvc-create.html#2-spring-mvc-%EC%B6%94%EA%B0%80


### references
* http://javaengine.tistory.com/313
* http://multifrontgarden.tistory.com/108
* https://nesoy.github.io/articles/2017-02/SpringMVC
* http://jhleed.tistory.com/75


---
# Tip. Spring Project Encoding 관련 설정 항목들

 * 기본적으로 IDE설정에서 모두 UTF-8로 변경



### server.xml

 * Connector 설정
   * UTF-8추가
 ```
 <Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" URIEncoding="UTF-8" />
 ```

 * GET방식 한글 깨짐
   * 이클립스 server.xml 에서 port=8080과 port=8009두개의 connector에 URIEncoding="UTF-8"을 추가한다



### web.xml
서블릿 클래스는 JSP 페이지와 달리 설치뿐만 아니라 등록을 하는 과정을 필요로 한다.

여기서 서블릿 클래스를 등록하는 곳의 이름을 Web application deployment descriptor라고 하는데 (줄여서 DD-Deployment Descriptor) 이 역할을 하는 것이 바로 web.xml이다. web.xml 파일은 웹 애플리케이션 디렉터리마다 딱 하나씩만 존재할 수 있다.

DD는 WAS 구동 시 /WEB-INF 디렉토리에 존재하는 web.xml을 읽어 웹 애플리케이션의 설정을 구성하기 위해 존재한다.


 * WAS서버가 알아야 하는 설정이 저장된다.
   * Dispatcher Servlet
   * 리스너
   * 필터
   * URL 패턴
   * error나 welcome 페이지

 * Servlet 3.0이상부터는 web.xml말고도 아래 방식으로 설정할 수 있다.
   * WebApplicationInitializer 인터페이스를 구현
   * AbstractAnnotationConfigDispatcherServletInitializer 클래스를 상속

 * 참고로
   * Spring MVC는 ServletContainerInitializer를 구현하고 있는 SpringServletContainerInitializer를 제공한다.
   * SpringServletContainerInitializer는 WebApplicationInitializer 구현체를 찾아 인스턴스를 만들고 해당 인스턴스의 onStartup 메소드를 호출하여 초기화한다.


 * filter 추가하는 예
 ```
 <filter>
                  <filter-name>encodingFilter</filter-name>
                  <filter-class>filter.EncodingFilter</filter-class>
                  <init-param>
                                   <param-name>encoding</param-name>
                                   <param-value>UTF-8</param-value>
                  </init-param>
</filter>
<filter-mapping>
                  <filter-name>encodingFilter</filter-name>
                  <url-pattern>*</url-pattern>
</filter-mapping>
 ```


 * POST 방식에서 한글 깨질 때도 web.xml에 filter class 를 등록해서 해결할 수 있다.



### contextConfigLocation(Application Context)
Spring에서 Application Context와 Servlet Context를 나누는 기준은 간단하다.

* Application Context
  - 공통 기능을 할 수 있는 Bean설정 (Datasource, Service 등..)
  - 각 Servlet에서 공유할 수 있는 Bean
* Servlet Context
  - Servlet 구성에 필요한 Bean 설정 (Controller, Interceptor, MappingHandler등..)


* Web Application 최상단에 위치하고 있는 Context
* Spring에서 ApplicationContext란 BeanFactory를 상속받고 있는 Context
* Spring에서 root-context.xml, applicationContext.xml 파일은 ApplicationContext 생성 시 필요한 설정정보를 담은 파일 (Bean 선언 등..)
* Spring에서 생성되는 Bean에 대한 IoC Container (또는 Bean Container)
* 특정 Servlet설정과 관계 없는 설정을 한다 (@Service, @Repository, @Configuration, @Component)
* 서로 다른 여러 Servlet에서 공통적으로 공유해서 사용할 수 있는 Bean을 선언한다.
* Application Context에 정의된 Bean은 Servlet Context에 정의 된 Bean을 사용할 수 없다.



Application Context 파일에서의 component-scan 설정 예시
```xml
<context:component-scan base-package="com.myapp.core, com.myapp.app">
    <!-- Component-scan대상에서 @Controller annotation Class는 제외한다. -->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```


### dispatcherServlet(Servlet Context)

 * dispatcher servlet이 초기화시에 관련된 전략 빈들(전략 패턴으로 구현된)을 Dispatcher Servlet에 주입한다.

Servlet Context 파일에서의 component-scan 설정 예시
```xml
<context:component-scan base-package="com.myapp.app" use-default-filters="false">
   <!-- Component-scan대상은 @Controller annotation Class만 scan한다. -->
   <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

use-default-filters 속성을 false로 처리 하였다.
use-default-filters 속성은 원래 default가 true인데,
@Compont Annotation(@Controller, @Service, @Repository등..) 의 클래스를 자동으로 Bean으로 등록해 주는 filter속성이다.
따라서 위의 필터를 false로 변경하고, scan할 대상에 대한 Annotation만 include-filter에 추가하면 된다.

##### 전략 빈(전략 패턴으로 구현된)
 * 전략 빈의 생성은 <mvc:annotation-driver/> 혹은 @EnableMvc를 통해 웹 컨테이너가 initialize되는 시점

 * 전략 빈 리스트
   * MultipartResolver
     + multipart 파일 업로드 요청에 대해 해석하여 변환
   * LocaleResolver
     + 웹 기반의 locale 결정 전략 인터페이스이다. 스프링에서는 Request, Session, Cookie 등을 기반으로 사용한 구현체들이 있다.
   * ThemeResolver
     + 웹 기반의 Theme 결정 전략 인터페이스. <Spring:theme> 태그를 사용한 경우 구현체 ThemeResolver로 Theme를 정의할 수 있다.
   * HandlerMapping
     + Request와 핸들러(메소드) 객체를 매핑을 정의하는 인터페이스이다. 일반적으로 사용하는 전략은 @RequestMapping 을 이용한 핸들러 매핑 전략이다.
   * HandlerAdapter
     + Spring-MVC의 코어 SPI(Service Provider Interface). Controller의 구현 전략 인터페이스이다. HandlerMapping이 핸들러(메소드)를 찾아주면, HandlerAdpter가 해당 컨트롤러의 핸들러에 전달하기 위한 Adapter라고 보면 된다.
	 + HandlerAdapter 인터페이스를 보면 ModelAndView 를 리턴하는 handle 이라는 메소드가 있다. 핸들러를 실행(handle)시켜 ModelAndView를 리턴하게끔 해준다고 생각하면 된다.
     + 스프링의 Controller 종류는 4가지가 있다. Servlet, Controller, HttpRequestHandler 구현체와 @Controller 어노테이션을 사용해서 POJO로 구현하는 방법이다.
     + 각 컨트롤러를 DispatcherServlet에 연결해주는 Adapter가 하나씩 있어야 하므로, handlerAdapter도 총 4개이다. 그 중 SimpleServletHandlerAdapter를 제외한 3개의 핸들러 아답터가 DispatcherServlet의 default 전략으로 설정되어 있다.
   * HandlerExceptionResolver
     + Handler 매핑이나, 컨트롤러 실행 도중 발생한 예외를 다루는 인터페이스이다.
	 + 디폴트로 AnnotationMethodHandlerExceptionResolver(@ExceptionHandler 처리), ResponseStatusExceptionResolver(발생한 예외를 HTTP status code로 변경), DefaultHandlerExceptionResolver(앞선 exceptionResolver가 처리하지 못한 예외상황을 처리한다.
	 + 메소드를 찾지 못해 발생하는 예외는 HTTP 404를 발생시키는 등의 로직 수행) 가 선언된다.
   * RequestToViewNameTranslator
     + 컨트롤러에서 뷰를 지정하지 않았을 때, Request를 참고하여 내부적으로 뷰 이름을 생성해준다.
   * ViewResolver
     + 컨트롤러에서 Request를 처리하고 생성하는 결과물에 대한 view 처리 전략 인터페이스이다.
	 + 디폴트는 InternalResourceViewResolver (컨트롤러가 리턴한 뷰 이름으로 실제 뷰 이름을 지정. prefix와 suffix를 조합하는 로직도 여기에 있다) 이다.
   * FlashMapManager
     + FlashMap 객체를 retrieve & save 하는 전략 인터페이스이다.
     + redirect 해야하는 경우에, 특정 데이터를 넘겨야 할 때가 있다. 이때 RedirectAttributes 를 이용하여 데이터를 넘길텐데 2가지 방법이 있다.
	 + addAttribute 와 addFlashAttribute 이다. addAttribute는 request 파라미터로 넘겨주는 방식이고, addFlashAttribute 는 FlashMap을 이용해 넘길 데이터를 세션에 저장하고 retrieve 하면 세션에서 바로 삭제한다.
	 + flashMap에 담은 내용은 세션에 저장되고 redirect되어 데이터를 받게되면 바로 삭제한다.
 * 모든 전략 인터페이스의 처리 과정은 주입된 빈에 의해 이루어진다. 커스터마이징이 필요한 부분은 인터페이스를 구현하여 스프링 빈으로 등록하면 된다.
 * 스프링 빈으로 등록만 해놓으면 DispatcherServlet 이 초기화 시점에 빈 주입이 된다.
 * 디폴트 빈은 <mvc:annotation-driven/> 혹은 @EnableWebMvc 를 사용할 때 생성된다.
 * 디폴트 빈 설정은 spring-mvc 프로젝트내 DispatcherServlet.properties 에 선언되어 있으니 확인해보면 된다.




 * 간략하게 MVC 프로세스를 간략히 정리해보면
 * 1. 배포서술자에 DispatcherServlet 등록한다.
 * 2. 웹 컨테이너가 뜰때 DispatcherServlet의 전략 빈들이 생성된다.
 * 3. 실제 사용자의 Request가 들어온다.
 * 4. 웹 컨테이너는 DispatcherServlet 객체를 생성하고 초기화 한다. (첫 호출시에만)
 * 5. 웹 컨테이너가 Request, Response 객체를 생성한다.
 * 6. 웹 컨테이너의 쓰레드 풀에서 쓰레드 하나를 할당하여 DispatcherServlet 실행.
 * 7. request uri에 알맞은 핸들러(메소드)를 가져온다. (실제 핸들러를 매핑하는 전략은 DispatcherServlet에 주입된 HandlerMapping 구현체에 의해)
 * 8. 핸들러를 실행시키기 위하여 핸들러 아답터를 가져온다. (핸들러 클래스 타입마다 적용시킬 아답터가 다르므로)
 * 9. 인터셉터의 preHandle 실행
 * 10. 실제 컨트롤러 핸들러(메소드) 실행  
 * 11. 인터셉터의 postHandle 실행
 * 12. 개발자가 설정한 View 에 알맞도록 렌더링을 실행하여 결과를 write
 * 13. 웹 컨테이너가 response를 보내주고 유저 스레드 반환


---
# Spring MVC 설정
```
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = { "kr.or.connect.webmvc.controller" })
public class WebMvcContexetConfiguration extends WebMvcConfigurerAdapter {
   .....
}
```


### jsp 파일 설정

 * UTF-8 설정
 ```
 <%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="UTF-8"%>
 ```


---
# 스프링 설정에 대해서

### XML based Configuration

 * 웹에서 XML based configuration으로 검색하면 스프링의 가이드가 나오는데, 도움이 되는거 같다.

### Annotation based Configuration


---
# Controller 관련 Spring MVC Annotation

### Spring MVC가 지원하는 Controller메소드 인수 타입

 * javax.servlet.ServletRequest
 * javax.servlet.http.HttpServletRequest
 * org.springframework.web.multipart.MultipartRequest
 * org.springframework.web.multipart.MultipartHttpServletRequest
 * javax.servlet.ServletResponse
 * javax.servlet.http.HttpServletResponse
 * javax.servlet.http.HttpSession
 * org.springframework.web.context.request.WebRequest
 * org.springframework.web.context.request.NativeWebRequest
 * java.util.Locale
 * java.io.InputStream
 * java.io.Reader
 * java.io.OutputStream
 * java.io.Writer
 * javax.security.Principal
 * java.util.Map
 * org.springframework.ui.Model
 * org.springframework.ui.ModelMap
 * org.springframework.web.multipart.MultipartFile
 * javax.servlet.http.Part
 * org.springframework.web.servlet.mvc.support.RedirectAttributes
 * org.springframework.validation.Errors
 * org.springframework.validation.BindingResult
 * org.springframework.web.bind.support.SessionStatus
 * org.springframework.web.util.UriComponentsBuilder
 * org.springframework.http.HttpEntity<?>
 * Command 또는 Form 객체


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


### Spring MVC가 지원하는 메소드 리턴 값

 * org.springframework.web.servlet.ModelAndView
 * org.springframework.ui.Model
 * java.util.Map
 * org.springframework.ui.ModelMap
 * org.springframework.web.servlet.View
 * java.lang.String
 * java.lang.Void
 * org.springframework.http.HttpEntity<?>
 * org.springframework.http.ResponseEntity<?>
 * 기타 리턴 타입




---
# 설정의 분리

 * Spring 설정 파일을 프리젠테이션 레이어쪽과 나머지를 분리할 수 있다.
 * web.xml 파일에서 프리젠테이션 레이어에 대한 스프링 설정은 DispathcerServlet이 읽도록 하고, 그 외의 설정은 ContextLoaderListener를 통해서 읽도록 한다.
 * DispatcherServlet을 경우에 따라서 2개 이상 설정할 수 있는데 이 경우에는 각각의 DispathcerServlet의 ApplicationContext가 각각 독립적이기 때문에 각각의 설정 파일에서 생성한 빈을 서로 사용할 수 없습니다.
 * 위의 경우와 같이 동시에 필요한 빈은 ContextLoaderListener를 사용함으로써 공통으로 사용하게 할 수 있습니다.
 * ContextLoaderListener와 DispatcherServlet은 각각 ApplicationContext를 생성하는데, ContextLoaderListener가 생성하는 ApplicationContext가 root컨텍스트가 되고 DispatcherServlet이 생성한 인스턴스는 root컨텍스트를 부모로 하는 자식 컨텍스트가 됩니다.
 * 참고로, 자식 컨텍스트들은 root컨텍스트의 설정 빈을 사용할 수 있습니다.





---
# Trouble Shooting

### Encoding

##### tomcat 기동중에 xml파일에서 UTF-8 인코딩 관련 오류

```
MalformedByteSequenceException: Invalid byte 1 of 1-byte UTF-8 sequence.
```

 * 한글 리소스가 아니고 단순히 xml파일에 있는 코멘트를 인식하지 못하는 것

 * 원인 파악이 안되서 시간을 많이 소비함

 * 결국엔 eclipse.ini 에 아래 옵션을 넣고 기동하였다. <= 이렇게 한 경우의 영향범위를 잘 알지 못함.
 ```
 -Dfile.encoding=UTF-8
 ```



---
# Tip. Spring Controller에서 Parameter를 받는 방식

### Request를 통해 받는 방법

```java
	@RestController
    class TempController {

        @GetMapping("/temp")
        String temp(HttpServletRequest request) {
        	String a = request.getParameter("a");
            String b = request.getParameter("b");

            return "none";
        }
    }
```

### Map으로 받는 방법

```java
	@RestController
    class TempController {

    	@GetMapping("/temp")
        String temp(@RequestParam Map<String, String> param) {
        	String a = param.get("a");
        	String b = param.get("b");

        	return "none";
    	}
    }
```

### @RequestParam을 통해 직접 매칭하는 방법

```java
	@RequestController
    class TempController {

    	@GetMapping("/temp")
        String temp(@RequestParam("a") String a, @RequestParam("b") int b) {

            return "none";
        }
    }
```

### Model Class로 받는 방법
 * HTTP 요청 파라미터를 전달받을 때 사용되는데, name에 해당하는 값을 입력
 * Parameter 타입을 int로 지정하면 자동으로 변환

```java
	@RestController
    class TempController {

    	@GetMapping("/temp")
    	String temp(Abc abc, Model model) {
    		//Model 스캔 시작
        	Map<String, Object> map = model.asMap();
        	for(String key : map.keySet()) {
        		System.out.println(key + " : " + map.get(key).toString());
        	}
        	//Model 스캔 종료

        	return "none";
    	}
    }
```

### @PathVariable로 받는 방법

 * View에 전달할 모델 데이터를 설정할 때 사용

 * 데이터의 양이 많은 경우 유용

```java
	@RestController
    class TempController {

        @GetMapping("/temp/{a}/{b}")
        String temp(@PathVariable("a") String a, @PathVariable("b") int b) {

            return "none";
        }
    }
```

### Model Class + @PathVariable 조합으로 받는 방법

```java
	@RestController
    class TempController {
    	@GetMapping("/temp/{a}/{b}")
        String temp(Abc abc) {
        	System.out.println(abc.getA());
            System.out.println(abc.getB());
            return "none";
        }

        @Data @ToString @Getter @Setter
        static class Abc {
        	String a;
            int b;
        }
    }
```


---
# Tip. forward , redirect

 * https://codediver.tistory.com/77

 스프링 프레임워크에서 컨트롤러의 메서드가 리턴하는 타입에 따라 포워딩과 리다이렉션 구현 방법을 간단히 기술한다. 단, 지원되는 resolver는 설정되어 있다고 가정 따로 언급하지 않는다.


### return String

```
 return "/member/login.do"; // 포워딩
 return "redirect:/member/login.do"; // 리다이렉션
```

 JavaCopy리다이렉트땐 'redirect:' 이후 꺽쇠(/)의 여부에 따라 클라이언트에 전달할 경로가 달라질 수 있다. 가령 Context 경로가 '/FO'이고 컨트롤러에 매핑된 경로의 최상단('/FO' 바로 다음)이 '/member'라고 했을 때 'redirect:member/login.do' 를 리턴하면 실제 전달되는 경로는 '/bo/member/member/login.do' 가 된다.


### return ModelAndView

```
 ModelAndView mv = new ModelAndView();
 mv.setViewName("/member/memberDetail");
 mv.addAttribute("msg", helloService.getMessage());
 //mav.addObject("msg", helloService.getMessage());
 return mav;

 // 혹은
 // req.setAttribute("msg", helloService);
 // return new ModelAndView("/member/memberDetail");JavaCopyModelAndView mv = new ModelAndView();
 mv.setViewName("redirect:/member/memberDetail.do?memberNumber=2");
 return mv;

 // 혹은
 // ModelAndView mv = new ModelAndView();
 // mv.setView(new RedirectView("/member/memberDetail.do?memberNumber=2", true));
 // return mv;
 ```

### ModelAndView

 ModelAndView는 포워드냐 리다이렉트냐 가 아니라 
 ModelAndView를 이용하여 포워드로도 리다이렉트로도 보낼 수 있는거다. 

* 포워드 
```
 ModelAndView mv = new ModelAndView();
 vm.setViewName("/notice/noticeList");
 vm.addAttribute("message", msgService.getMessage());
 return vm;

 or 

 req.setAttrebute("message", msgService.getMessage());
 return new ModelAndView("/notice/noticeList");
```

* 리다이렉트
```
ModelAndView mv = new ModelAndView();
vm.setViewName("redirect:/notice/noticeList.do?noticeNumber=2");
return vm;

or

ModelAndView mv = new ModelAndView();
vm.setView(new RedirectView("/notice/noticeList.do?noticeNumber=2",true));
return vm;
```
