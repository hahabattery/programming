---
layout: default
title: Overview
grand_parent: Springs
parent: Spring Cloud
nav_order: 1
---



# Spring Cloud Netflix
Netflix OSS(Open Source Software) 와 스프링에 통합을 지원한다.

Netflix Zuul 대신 Spring 에서 권장하는 gateway 서비스이다.

Spring Boot 2.4.x 이상부터 Zuul 1 을 사용할 수 없으며 Zuul 2 와 통합은 지원하지 않는다.

Spring Boot 2.6.x, Spring Cloud 2021.0.0 기준 Spring Cloud Netflix Eureka 만 남아있다.


[Replacements, Spring Blog](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now)

[The Future of Spring Cloud Microservices After Netflix Era](https://piotrminkowski.com/2019/04/05/the-future-of-spring-cloud-microservices-after-netflix-era/)

# Spring Cloud Gateway

Spring 5, Spring Boot 2, Project Reactor 를 기반으로 API Gateway 를 구축을 지원한다.

> spring-mvc, spring-data, spring-security 등 동기방식의 프로젝트들과 함께 실행하면 문제가 발생할 수 있다.

* Spring Security 관련된 이슈들
  + SecurityWebFilterChain bean을 생성해서 사용하고, 관련 설정을 해줘야 한다.
  + https://www.baeldung.com/spring-webflux


* stack overflow   
  + https://stackoverflow.com/questions/69479286/spring-cloud-with-spring-security-not-working
  + https://stackoverflow.com/questions/67268107/cannot-access-javax-servlet-filter-error-when-using-spring-security-with-sprin/67269039#67269039
  + https://stackoverflow.com/questions/51234311/can-i-implement-websecurityconfigureradapter-on-spring-webflux
  + https://stackoverflow.com/questions/47354171/spring-webflux-custom-authentication-for-api



* Spring Security for reactive applications
  + https://docs.spring.io/spring-security/site/docs/5.4.6/reference/html5/#reactive-applications
  + https://docs.spring.io/spring-security/site/docs/5.4.6/reference/html5/#jc-webflux <= "Minimal WebFlux Security Configuration"




![](./images/springs/spring-cloud/SpringCloudGatewayOverview-1.jpg)


클라이언트에서 들어온 요청은 Gateway Handler Mapping을 통해 요청 경로와 일치 여부를 판단하게 되고, Gateway Web Handler에서 요청과 관련된 필터 체인을 통해 요청이 전송되게 됩니다.

이후 적용되는 Filter를 통해 요청 또는 응답에 필요한 전처리, 후처리를 할 수 있으며 Proxy Filter는 프록시 요청이 처리될 때 수행됩니다.


* 이해하기 쉬운 설명
  + https://wildeveloperetrain.tistory.com/207
  + https://velog.io/@csh0034/Spring-Cloud-Gateway-JWT-%EC%9D%B8%EC%A6%9D


# Examples
https://github.com/spring-cloud-samples <- 조금 오래된 예제들인거 같다.

https://github.com/piomin/sample-spring-microservices-new <- master branch가 SpringBoot 3.0으로 작업되있고, netflix oss도 제거되있어서, 이게 조금 최신인 편
