---
layout: default
title: Thymeleaf
parent: Template Engines
nav_order: 10
---


# Spring 과의 Integration
For docs on Spring MVC and Thymeleaf integration, see this link
- https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html

For docs on Spring Security and Thymeleaf integration, see this link
- https://www.thymeleaf.org/doc/articles/springsecurity.html


### pom.xml(Thymeleaf pom entries)
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		
		<dependency>
			<groupId>org.thymeleaf.extras</groupId>
			<artifactId>thymeleaf-extras-springsecurity5</artifactId>
		</dependency>
```

### Spring Boot Auto-Configuration
When Spring Boot finds Thymeleaf dependency in the Maven POM file, it automatically configures Thymeleaf template engine. 
No need to manually configure Thymeleaf in our code since it is auto-configured by Spring Boot.


# utility

### null check와 빈 문자열을 체크

```thymeleaf
<div th:if="${not #strings.isEmpty(examData)}"> ... </div>
```
#strings utility의 isEmpty()를 사용하면 null이거나 빈 문자열일 때 true를 반환한다. trim() 메소드가 적용되기 때문에, 빈 줄이나 공백도 같이 걸러지게 된다.

#strings utility 에는 String 객체를 위한 다양한 method 들이 있다.
[#strings : utility methods for String objects](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#strings)

