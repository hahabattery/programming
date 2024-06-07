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



# Text and Escape
< , >와 같은 특수문자를 HTML 엔티티라고도 표현한다

### th:text [[${...}]]

```
<span th:text="${data}">Example</span>
```

위와 같이 th:text를 사용하면 Example -> ${data} 값으로 치환되게 된다
만약 치환하지 않고 콘텐츠 안에서 직접 출력하고 싶다면 아래와 같이 작성하면 된다

```
<span>[[${data}]]</span>
```

Hello <b>Spring!</b>와 같은 값이 ${data}로 넘어왔다고 가정해보자
 
개발자의 의도대로라면 Spring!이 bold처리가 되어 굵게 표시가 되어야 하겠지만
타임리프가 이스케이프를 하게 되면 아래와 같이 출력이 된다
 
웹브라우저 : <b>Spring!</b>
페이지 소스: &lt;b&gt;Spring!&lt;/b&gt;

### th:utext [(${...})]
unescape를 해야 하는 상황이라면 th:utext와 [(${...})]를 사용해서 해결할 수 있다.

```
<span th:utext="${data}"></span>
```

```
<span>[(${data})]</span>
```
