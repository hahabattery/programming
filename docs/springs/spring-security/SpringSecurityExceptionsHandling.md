---
layout: default
title: Spring Security Exceptions
grand_parent: Springs
parent: Spring Security
nav_order: 4
---

Spring Security에서 예외처리관련 내용을 정리

---
# 참조

[인증 / 인가 API의 예외처리](https://coder-in-war.tistory.com/entry/Spring-Security-05-%EC%9D%B8%EC%A6%9D%EC%9D%B8%EA%B0%80-API%EC%9D%98-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC)

 * [Spring Security에서 설정하는 accses denied 페이지](https://zgundam.tistory.com/60)
   +

---
# ExceptionTranslationFilter
이 필터는 보호된 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달하는 역할을 한다.

FilterSecurityInterceptor(필터중에 순서가 맨 마지막에 위치)가 이를 호출한다.

ExceptionTranslationFilter는 아래 2가지 예외를 처리한다.

 * AuthenticationException (인증 예외 처리)
   + AuthenticationEntryPoint 호출
     + 로그인 페이지 이동
     + 401 오류 코드 전달
   + 인증 예외가 발생하기 전의 요청정보를 저장
     + **RequestCache** : 사용자의 이전 요청 정보를 세션에 저장하고 이를 꺼내오는 캐시 메커니즘을 제공
     + **SavedRequest** : 사용자가 요청했던 request의 파라미터 값들, 그 당시의 헤더값 들이 저장
     + 위의 2개는 모두 인터페이스이며 세부 기능을 하는 구현체가 존재한다.
 * AccessDeniedException (인가 예외 처리)
   + AccessDeniedHandler에서 예외 처리하도록 제공
   + 얘도 인터페이스이며 세부 기능을 하는 구현체가 존재한다.

![](imgs/SpringSecurity-예외처리-ExceptionTranlsationFilter.png)

유저가 /user 자원 접근을 시도하고 있다면, FilterSecurityInterceptor에서 엄밀히 말해서는 익명사용자이기 때문에 인증 예외가 아니라, 인가 예외로 빠지게 된다.

이 ExceptionTranlsationFilter가 인가 예외로 보내긴 하지만, AccessDeniedHandler로 곧바로 보내지 않고, **HttpSessionRequestCache** 로 보낸다.

따라서, 쉽게 생각해서 인증 예외의 과정으로 보내어진다고 보면된다.

인증예외파트에서는 두 가지 일을 하게 된다.

 * 인증 실패 이후 처리
   + Security Context안에 있는 인증 객체를 null로 만든다.
   + **AuthenticationEntryPoint** 에서 다시 인증할 수 있도록 로그인 페이지로 보내버리고
 * 사용자의 요청관련 정보 저장
   + 예외가 발생하기 이전의 유저정보를 저장한다.
   + 이 정보는 **DefaultSavedRequest** 객체 안에 저장되고 이 객체는 Session에 저장한다.
   + 그 역활을 **HttpSessionRequestCache** 클래스가 수행한다.



만약에 유저가 /user 페이지가 아니라, /admin 자원에 접근하려고 한다면 인가 예외가 발생한다.

ExceptionTranslationFilter는 **AccessDeniedException**이 발생하고 AccessDeniedHandler를 호출해서 그 안에서 후속처리를 한다.

(보통은 /denied 페이지를 redirect한다. 또는 이 자원에 다시금 접근할 수 있도록 조정할 수 있는 처리를 하게 된다.)

---
# 예외 처리 과정

![](imgs/SpringSecurity-예외처리-예외처리과정소스.png)

그리고 이때 sendRedirect 할 때는 Spring-Security에서 제공하는 페이지가 아니기 때문에 직접 페이지를 만들어줘야 한다.



![](imgs/SpringSecurity-예외처리-흐름.png)
