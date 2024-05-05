---
layout: default
title: Overview
grand_parent: Springs
parent: Spring Security
nav_order: 1
---



# 아래 내용에 대해서 찾아보자

same-site 쿠키 옵션?

CSRF 적용 (https://coder-in-war.tistory.com/entry/Spring-Security-06-CSRF-Filter)


# 리소스

* OAuth 사용시의 처리 확인 필요.
  + https://parkhyeokjin.github.io/spring/2019/11/19/Spring-OAuth.html

* CSRF - redis
  + https://github.com/spring-guides/tut-spring-security-and-angular-js/issues/156
  + https://stackoverflow.com/questions/38160916/csrf-token-in-redis-spring-session

* multi login page
  + https://www.baeldung.com/spring-security-two-login-pages

[Spring Boot - (6) 민감 정보 숨기기(DB id, password, etc.)](https://4urdev.tistory.com/48)

# 최근에 보던 글

 * Spring Security의 Definitive Tutorial
   + [Spring Security Tutorial](https://www.javadevjournal.com/spring-security-tutorial/)


 * https://www.udemy.com/course/spring-security-with-json-web-token-and-refresh-token/
   + https://github.com/bezkoder/spring-boot-refresh-token-jwt



 * [spring security 파헤치기 (구조, 인증과정, 설정, 핸들러 및 암호화 예제, @Secured, @AuthenticationPrincipal, taglib)](https://sjh836.tistory.com/165)

 * [Spring Boot + Vue.js: Authentication with JWT & Spring Security Example](https://www.bezkoder.com/spring-boot-vue-js-authentication-jwt-spring-security/)


 * [스프링 시큐리티] 2. Form Login 인증 방식 파악 UsernamePasswordAuthenticationFilter, AnthPathRequestMatcher
   * https://whitepro.tistory.com/473?category=1058993
   * https://velog.io/@eldehd9898/Spring-Security-%EC%A0%95%EC%88%98%EC%9B%90









# jsp

### AuthenticationToken 정보 조회

pom.xml에 spring-security-taglibs 패키지를 추가합니다.

```xml
<dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-taglibs</artifactId>
</dependency>
```

jsp 상단에 추가한 taglib를 import합니다.
```jsp
<%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec" %>
```
그러면 taglib를 이용해 다음과 같이 AuthenticationToken 정보를 조회할 수 있습니다.

```jsp
<sec:authentication property="principal.userId"/>
```








# Method Level Security
API path / URL 을 이용해서 authorization 처리가 가능하다.
또한, service layer나 reposository layer에도 적용할 수 있는 방법을 제공한다.


> Method level security will also helps authorization rules even in the non-web applications where we will not have any endpoints.


 * @EnableGlobalMethodSecurity
   + configuration 클래스에 사용하면 method level security가 적용된다.


** 설정 예 **
```
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class ProjectSecurityConfig {
  ...
}
```

 * prePostEnabled enable Spring Security의 @PreAuthorize @PostAuthorize
 * securedEnabled enable @Secured
 * jsr250Enabled enable @RoleAllowed






# 스프링시큐리티 관련 이해하기 쉬운 글 모임


### spring security 공식 매뉴얼

 * https://godekdls.github.io/Spring%20Security/contents/ <= 한글번역판, 바로 아래는 영문
 * https://docs.spring.io/spring-security/site/docs/5.3.2.RELEASE/reference/html5/
 * https://spring.io/guides/tutorials/spring-boot-oauth2/


### 스프링시큐리티 튜토리얼
 * https://catsbi.oopy.io/c0a4f395-24b2-44e5-8eeb-275d19e2a536
   + 강추
 * https://coding-start.tistory.com/153
   + 스프링시큐리티 처리흐름이 꼼꼼이 잘 적혀 있음
   + 하단의 "Java Config 기반 코드 설명" 부분도 유용함.
 * [Example Of Multiple Login Pages With Spring Security And Spring Boot](https://bartslota.com/example-of-multiple-login-pages-wi/)

### OAuth 관련

 [Spring Boot OAuth2 Social Login with Google, Facebook, and Github - Part 1](https://www.callicoder.com/spring-boot-security-oauth2-social-login-part-1/)

 [Spring Boot OAuth2 Social Login with Google, Facebook, and Github - Part 2](https://www.callicoder.com/spring-boot-security-oauth2-social-login-part-2/)


 * [Spring Boot + Kakao Login API 연동 (기초)](https://momentjin.tistory.com/144)
   + => Kakao OAuth API관련한 내용.
   + spring security의 동작을 이해하기 쉽게 적음.
 * [Spring Boot + Kakao Login API 연동 (심화)](https://momentjin.tistory.com/146)

 * https://github.com/Candivel/KakaoLogin
   + 이건 내용이 잘 정리되있긴 한데, 안드로이드임 크흠

 * [Spring Boot OAuth2 Login with GitHub Example](https://www.codejava.net/frameworks/spring-boot/oauth2-login-with-github-example)
   + 2021년도 글 그나마 최신?

### 스프링 시큐리티 관련 글(ppaksang.tistory)
실제 프로젝트 진행하면서 일지처럼 작성하였음.

 * Spring Security 에서 JWT 를 통한 인증/인가 수행하기
   + https://ppaksang.tistory.com/12
 * Application Architecture(인증/인가, Header 값 분리, CORS 허용)
   + https://ppaksang.tistory.com/13
 * Access Token 과 Refresh Token 을 어디에 저장하고 어떻게 교환해야 할까?
   + https://ppaksang.tistory.com/16



### AuthenticationFilter 에서 JWT토큰을 생성하는 예제

 * https://wonyong-jang.github.io/spring/2020/08/15/Spring-Security-Database-Authentication.html
 * https://wonyong-jang.github.io/spring/2020/08/17/Spring-Security-JWT.html

 * https://minholee93.tistory.com/category/Spring/Security



---
# 스프링 시큐리티 테스트관련한 내용

 * https://tecoble.techcourse.co.kr/post/2020-09-30-spring-security-test/
   + 스프링 시큐리티에서 지원하는 여러가지 테스트용 어노테이션에 대한 내용

 * https://bartslota.com/example-of-multiple-login-pages-wi/
   + 스프링 시큐리티 테스트시에 매우 도움되는 글!



---
# 동일 세션 차단 (최대 사용 기기 제한)
검색 키워드 - "세션 접속 차단"

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=overpjh&logNo=70163959312
https://ddolcat.tistory.com/2036

[Control the Session with Spring Security | Baeldung ](https://www.baeldung.com/spring-security-session)

[Spring Security Session – How to Control Session with Spring Security | javadevjournal](https://www.javadevjournal.com/spring-security/spring-security-session/)
=> 세션 관리가 엄청 복잡하다...

https://hojak99.tistory.com/613 <= 이건 JWT를 이용한 개수 제한에 대한 검토 내용

### 구현예 1

https://syaku.tistory.com/329

https://github.com/syakuis/springframework/tree/duplication-login-filter

https://okky.kr/articles/378292

https://ziponia.github.io/2019/05/19/spring-boot-security-options.html



---
# Spring Security 설정

server.servlet.session.timeout은 deprecated되었고, spring.session.timeout이 우선적으로 적용됨.


# The type WebSecurityConfigurerAdapter is deprecated
Spring Security 5.7.1부터, 혹은 Spring Boot 2.7.0부터  WebSecurityConfigurerAdapter가 deprecated되었다.

https://www.codejava.net/frameworks/spring-boot/fix-websecurityconfigureradapter-deprecated

### 변경사유
 * Component 생성 방식으로 보안 설정을 하도록 권장하고 있다.
   + 위 링크에 구체적인 예가 있다.
