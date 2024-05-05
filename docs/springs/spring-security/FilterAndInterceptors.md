---
layout: default
title: Filter And Interceptor
grand_parent: Springs
parent: Spring Security
nav_order: 4
---

# Filter / Interceptor

[필터(Filter) vs 인터셉터(Interceptor) 차이 및 용도 - (1)](https://mangkyu.tistory.com/173)

[필터(Filter)가 스프링 빈 등록과 주입이 가능한 이유(DelegatingFilterProxy의 등장) - (2)](https://mangkyu.tistory.com/221)
>등장 배경: 필터에서 다른 스프링 빈의 주입이 필요해짐
>DelegatingFilterProxy의 등장 이전: 스프링 빈으로 등록 및 다른 빈 주입이 불가능했음
>DelegatingFilterProxy의 등장 이후: DelegatingFilterProxy를 통해 스프링 빈으로 등록 및 다른 빈 주입이 가능해짐
>SpringBoot의 등장 이후: 웹서버를 직접 관리하면서 DelegatingFilterProxy조차 필요없게됨

[Spring Filter와 Interceptor](https://jaehun2841.github.io/2018/08/25/2018-08-18-spring-filter-interceptor/#4-Component-WebFilter-Order-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%95%84%ED%84%B0-%EB%93%B1%EB%A1%9D)

[A Review of Filters](https://docs.spring.io/spring-security/reference/servlet/architecture.html)


![](./imgs/filter-intercepter-table.png)

**Request/Response 객체 조작 가능 여부**

필터는 Request와 Response를 조작할 수 있지만 인터셉터는 조작할 수 없다. 여기서 조작한다는 것은 내부 상태를 변경한다는 것이 아니라 다른 객체로 바꿔친다는 의미이다. 이는 필터와 인터셉터의 코드를 보면 바로 알 수 있다.

필터가 다음 필터를 호출하기 위해서는 필터 체이닝(다음 필터 호출)을 해주어야 한다. 그리고 이때 Request/Response 객체를 넘겨주므로 우리가 원하는 Request/Response 객체를 넣어줄 수 있다. NPE가 나겠지만 null도 세팅가능.
```java
public MyFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        // 개발자가 다른 request와 response를 넣어줄 수 있음
        chain.doFilter(request, response);       
    }

}
```


하지만 인터셉터는 처리 과정이 필터와 다르다. 디스패처 서블릿이 여러 인터셉터 목록을 가지고 있고, for문으로 순차적으로 실행시킨다. 그리고 true를 반환하면 다음 인터셉터가 실행되거나 컨트롤러로 요청이 전달되며,
false가 반환되면 요청이 중단된다. 그러므로 우리가 다른 Request/Response 객체를 넘겨줄 수 없다. 그리고 이러한 부분이 필터와 확실히 다른 점이다.
하지만, Request/Response 객체말고 객체가 내부적으로 갖는 값은 조작할 수 있다.
```java
public class MyInterceptor implements HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // Request/Response를 교체할 수 없고 boolean 값만 반환할 수 있다.
        return true;
    }

}
```


### 필터(Filter)의 용도 및 예시
* 공통된 보안 및 인증/인가 관련 작업
* 모든 요청에 대한 로깅 또는 감사
* 이미지/데이터 압축 및 문자열 인코딩
* Spring과 분리되어야 하는 기능

필터에서는 기본적으로 스프링과 무관하게 전역적으로 처리해야 하는 작업들을 처리할 수 있다.

대표적으로 보안 공통 작업이 있다. 필터는 인터셉터보다 앞단에서 동작하므로 전역적으로 해야하는 보안 검사(XSS 방어 등)를 하여 올바른 요청이 아닐 경우 차단을 할 수 있다. 그러면 스프링 컨테이너까지 요청이 전달되지 못하고 차단되므로 안정성을 더욱 높일 수 있다.

또한 필터는 이미지나 데이터의 압축이나 문자열 인코딩과 같이 웹 애플리케이션에 전반적으로 사용되는 기능을 구현하기에 적당하다. Filter는 다음 체인으로 넘기는 ServletRequest/ServletResponse 객체를 조작할 수 있다는 점에서 Interceptor보다 훨씬 강력한 기술이다.

대표적으로 필터(Filter)를 인증과 인가에 사용하는 도구로는 SpringSecurity가 있다. SpringSecurity의 특징 중 하나는 Spring MVC에 종속적이지 않다는 것인데, 필터 기반으로 인증/인가 처리를 하기 때문이다.



### 인터셉터(Interceptor)의 용도 및 예시
* 세부적인 보안 및 인증/인가 공통 작업
* API 호출에 대한 로깅 또는 감사
* Controller로 넘겨주는 정보(데이터)의 가공

인터셉터에서는 클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업들을 처리할 수 있다.

대표적으로 세부적으로 적용해야 하는 인증이나 인가와 같이 클라이언트 요청과 관련된 작업 등이 있다. 예를 들어 특정 그룹의 사용자는 어떤 기능을 사용하지 못하는 경우가 있는데, 이러한 작업들은 컨트롤러로 넘어가기 전에 검사해야 하므로 인터셉터가 처리하기에 적합하다.

또한 인터셉터는 필터와 다르게 HttpServletRequest나 HttpServletResponse 등과 같은 객체를 제공받는 구조이기 때문에, 객체 자체를 조작할 수는 없다. 대신 해당 객체가 내부적으로 갖는 값은 조작할 수 있으므로 컨트롤러로 넘겨주기 위한 정보를 가공하기에 용이하다. 예를 들어 사용자의 ID를 기반으로 조회한 사용자 정보를 HttpServletRequest에 넣어줄 수 있다.

그 외에도 우리는 다양한 목적으로 API 호출에 대한 정보들을 기록해야 할 수 있다. 이러한 경우에 HttpServletRequest나 HttpServletResponse를 제공해주는 인터셉터는 클라이언트의 IP나 요청 정보들을 포함해 기록하기에 용이하다.

**실제 사용예**
인터셉터는 주로 세션에 대한 체크, 인증 처리 등에 사용한다. 대부분의 어뷰징에 대한 처리를 인터셉터 단에서 처리하는 경우도 있다.
또한 특정 상품 진입 시, 권한 체크, 제약 조건 체크들도 인터셉터 단에서 처리





# Interceptor VS 스프링 시큐리티

### okky
 * https://okky.kr/articles/638119
> Interceptor : 특정 URI로 요청했을 때 컨트롤러로 가는 요청을 가로챔
>
> spring security : 인증,권한,보안 기능을 제공하는 Spring의 하위 프레임워크
>
> **스프링 시큐리티는 필터와 인터셉터 여러개로 이루어진 모듈이기 때문에,** 그 안에 무슨 역할을 하는 필터와 인터셉터가 있는지 알아보면 이해가 됨.
>
> security를 쓴다는 것은 spring에서 제공하는 인증관련 옵션을 사용한다고 보시면 될 것 같습니다.
>
> 개발자 입장에서 난이도 높은 보안 관련 코딩을 직접하지 않아도 된다는 장점이 있지만 사용하기에 난이도가 있는 편입니다.
>
> 예전에 했었던 프로젝트들이 인터셉터를 썼던거는 인증관련 처리를 개발자가 다 알아서 하기 위해서(자유도가 높다?)였던거 같다.
>
> Spring Security 가 양과 관련 자료가 많아서 러닝커브가 있기 때문에, 인터셉터를 사용해서

### inflearn

 * https://www.inflearn.com/questions/614369
> 실무에서도 시큐리티를 사용하는 경우도 있고, 필터나 인터셉터를 이용해서 직접 구현하는 경우도 있습니다.
>
> 전체 개발자들의 스프링 시큐리티 이해도가 높다면 스프링 시큐리티를 사용하는 것이 편하고, 그렇지 않고 인증 인가 부분이 복잡하지 않다면 직접 구현해서 사용하는 것도 좋은 방법입니다.





 * [Authentication/Authorization 기능 구현(1) - interceptor vs spring security](https://soon-devblog.tistory.com/4)

 * [Authentication/Authorization 기능 구현(2) - interceptor vs spring security](https://soon-devblog.tistory.com/5)

 * [HttpSession은 "언제" 만들어질까?](https://soon-devblog.tistory.com/2)

 * [Spring-security authorizeRequest 동적으로 설정하기](https://soon-devblog.tistory.com/7)
   + => DB에서 접근가능한 권한을 관리함.



# Filter

### GenericFilterBean



### OncePerRequestFilter
filter가 한번만 실행되어야 할 떄

![](imgs/OncePerRequestFilter-1.png)

1) api 1 호출 request
2) first , second filter
3) api1 controller 요청받음. 요청 처리 후 , api2로 sendRedirct 함.
4) 다시 first , second filter를 탐
5) api2 controller 요청받음. 요청 처리후 response

이런 흐름을 보여주고 있다. 여기서 문제점은 실제 요청자는 요청을 한번만 했을뿐인데, 흐름상 요청을 두번한것과 같은 형태가 되어 있다.


first 와 second 의 필터가 어떤것이냐 하는 상황에 따라 다르겠지만 간단히 인증처리를 하는 filter라고 생각을 한다면 요청자의 요청에 대한 인증처리는 한번만 하면 되는것인데


흐름상 무조건 두번을 또는 redirect를 몇번을 시키냐에 따라 여러번 인증처리 또는 불필요한 처리를 반복하게 된다.
이런 문제를 해결하기 위해 OncePerRequestFilter 라는 것이 나왔다.

>We expect that as soon as a request hits your project, you should authenticate and authorize it once.
>Then, if everything seems fine, this request and any other request from this context can be allowed to hit your APIs without the need to go through filters again.
>OncePerRequestFilter makes sure of it that this authentication process happens only once. If we don't use this, whenever we internally make a request to some other API in our project, the same authentication will happen again as all our APIs are having the same security filter
>
>A common use-case is in Spring Security, where authentication and access control functionality is typically implemented as filters that sit in front of the main application servlets.
>When a request is dispatched using a request dispatcher, it has to go through the filter chain again (or possibly a different one) before it gets to the servlet that is going to deal with it. The problem is that some of the security filter actions should only be performed once for a request. Hence the need for OncePerRequestFilter over GenericFilterBean.


# Interceptor

Interceptor 동작 방식
1. 외부로 부터 요청이 들어오면 DispatcherServlet에서 요청을 처리한다.
2. DispatcherServlet의 doDispatch() 메소드에서 getHandler() 메소드로 HandlerExecutionChain를 호출 한다.
  (정확히는 RequestMappingHandlerAdapter의 HandlerExecutionChain)
3. getHandler() 메소드 내부에는 getHandlerInternal() 메소드로 handler를 가져오는 부분이 있다.
  이 부분이 바로 요청 URL과 매칭하는 Controller 메소드를 찾아내는 부분이다.
4. 그 다음 getHandlerExecutionChain() 메소드에서 요청 메소드의 URL에 대해 이미 등록 된 interceptor 들의 url-pattern들과 매칭 되는 interceptor 리스트를 추출한다.
5. 추출 된 interceptor들에 대해 preHandle() 메소드를 실행 시킨다.
  (preHandle() 메소드의 리턴 타입은 boolean인데 false가 리턴 되는 경우에는 Controller 메소드를 실행 하지 않는다.)
6. 그 다음 요청 URL에 맞는 Controller 메소드를 실행 시킨다.
7. 메소드 작업이 끝난 뒤 추출 된 interceptor들에 대해 postHandle() 메소드를 실행 시킨다.



# Filter 설정

1. web.xml 등록 방식
```xml
<filter>
    <filter-name>testFilter</filter-name>
    <filter-class>com.example.springstudy.filter.TestFilter</filter-class>
</filter>
<filter>
    <filter-name>testFilter2</filter-name>
    <filter-class>com.example.springstudy.filter.TestFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>testFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <!-- url-pattern 대신 Servlet을 지정할 수도 있다. -->
    <servlet-name>testServlet</servlet-name>
</filter-mapping>
<filter-mapping>
    <filter-name>testFilter2</filter-name>
    <url-pattern>/*</url-pattern>
    <!-- url-pattern 대신 Servlet을 지정할 수도 있다. -->
    <servlet-name>testServlet</servlet-name>
</filter-mapping>
```



2. FilterRegistration Bean을 정의하여 추가할 Filter를 정의
```java
package com.example.springstudy.config;

import com.example.springstudy.filter.TestFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WebApplicationFilterConfig {

    @Bean
    public FilterRegistrationBean testFilterRegistration() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new TestFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setName("Test-Filter");
        filterRegistrationBean.setOrder(1);

        return filterRegistrationBean;
    }

    @Bean
    public FilterRegistrationBean testFilter2Registration() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new TestFilter2());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setName("Test-Filter2");
        filterRegistrationBean.setOrder(2);

        return filterRegistrationBean;
    }
}
```
Filter chain의 실행 순서는 Filter1(Prehandle) -> Filter2(Prehandle) -> Filter2 (Posthandle) -> Filter1 (Posthandle) 순으로 실행 된다.



3. AbstractAnnotationConfigDispatcherServletInitializer에서 getServletFilter에 추가
※ 이 방식은 Spring Boot 환경에서 동작하지 않았다. (Embedded Tomcat이어서 그런가? 잘 모르겠다.)

```java
package com.example.springstudy.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

import javax.servlet.Filter;

@Configuration
public class WebInitializerConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Filter[] getServletFilters() {
        //추가할 필터 리스트를 추가한다.
        return new Filter[]{new TestFilter(), new TestFilter2()};
    }

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[0];
    }

    @Override
    protected String[] getServletMappings() {
        return new String[0];
    }
}
```



4. @Component, @WebFilter, @Order 어노테이션을 이용한 필터 등록
```java
package com.example.springstudy.filter;

import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@Component
@WebFilter(
        description = "1번째 필터",
        urlPatterns = "/*",
        filterName = "Test-Filter1"
)
@Order(2)
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("start testFilter1");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("finish testFilter1");
    }

    @Override
    public void destroy() {

    }
}
```





# Interceptor 설정

1. Servlet-context.xml에 등록

```xml
<mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/client"/>
            <mvc:mapping path="/client/test1"/>
            <bean id="testInterceptor"
                  class="com.example.springstudy.interceptor.TestInterceptor"/>
        </mvc:interceptor>
</mvc:interceptors>
```

2. WebMvcConfigurationSupport 이용하여 등록
```java
package com.example.springstudy.config;

import com.example.springstudy.interceptor.TestInterceptor;
import com.example.springstudy.interceptor.TestInterceptor2;
import com.example.springstudy.resolver.ClientIpArgumentResolver;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

import java.util.List;

@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {

    /**
    * addInterceptors 메소드를 통해 Interceptor 등록
    */
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);
        //String... 타입으로 여러개 지정 가능
        registry.addInterceptor(new TestInterceptor())
                .addPathPatterns("/client/test1", "/client/help");
        //List 타입으로 여러개 지정 가능
        registry.addInterceptor(new TestInterceptor2())
                .addPathPatterns(Lists.newArrayList("/client", "/client/test1"));
    }
}
```
