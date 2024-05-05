---
layout: default
title: Spring Boot JSP
grand_parent: Springs
parent: Spring Boot
nav_order: 4
---

Spring Boot + JSP 구축방법을 정리

오래전에 만든 JSP프로그램을 스프링/스프링부트로 마이그레이션 하는 경우에 많이 사용됨.



# FROM
 * [Spring Boot에서 JSP 사용하기](https://lelecoder.com/81)

 * [Spring - JSP를 사용하는 스프링 부트 (Spring Boot) 프로젝트 만들기](https://7942yongdae.tistory.com/115)

 * 비슷한 다른 글 나중에 확인할 것
   + [실무에서 사용하는 React + SpringBoot 프로젝트 만들기 with Gradle](https://7942yongdae.tistory.com/136)
   + [vuejs를 사용하는 스프링 부트 프로젝트 만들기](https://7942yongdae.tistory.com/135)
   + [스프링 부트에서 오류 페이지를 정의하고 사용하는 방법](https://7942yongdae.tistory.com/140)


---
# spring initializr에서 프로젝트 생성
**먼저 https://start.spring.io 에서 Spring Web을 디펜던시로 생성해서 다운받는다.**


---
# JSP 컴파일을 위한 라이브러리를 추가

gradle
```
implementation "org.apache.tomcat.embed:tomcat-embed-jasper"
```

maven
```
<dependency>
  <groupId>org.apache.tomcat.embed</groupId>
  <artifactId>tomcat-embed-jasper</artifactId>
  <scope>provided</scope>
</dependency>
```


---
# JSP를 사용하기 위한 프로젝트 구조로 변경
스프링 부트에서 Thymeleaf를 사용하면 다운로드한 그대로 프로젝트 구조를 사용해도 문제가 없지만, 별도의 템플릿 엔진이나 JSP를 사용하기 위해서는 webapp와 WEB-INF 폴더를 포함한 구조로 프로젝트 구성을 변경해야한다.

webapp 폴더는 스프링에서 웹을 정의하는 root 폴더이며, WEB-INF는 J2EE 사양에 따라 정의된 표준 폴더 구조 웹 애플리케이션과 폴더 구조와 관련된 내용은 이 [링크](https://docs.oracle.com/cd/E13222_01/wls/docs90/webapp/configurewebapp.html)를 확인해서 보시면 더 자세하게 설명되어있음. 즉 스프링 부트를 이용해 스프링 프로젝트를 만들고 JSP를 사용하고 싶다면 webapp 폴더와 WEB-INF 폴더를 만들어야 한다.

![](./img/springboot-jsp-directory.png)

위에서 제시한 구조와 같이 프로젝트 폴더 구조를 변경하시면 웹 프로젝트의 기본 폴더 구조를 적용하게 된다. 예시 구조에서는 파일 구분을 위해 WEB-INF 폴더 밑에 view 폴더를 추가하였는데, view 폴더는 JSP 파일들만 넣어서 사용하기 위해 정의한 구조로, 개인 혹은 프로젝트의 성격에 따라 사용하면 된다.


---
# JSP를 사용하기 위한 ViewResolver 설정 변경

스프링 부트 프로젝트 구조가 변경이 일어났기 때문에 스프링 부트에서 사용하는 뷰 리졸버(ViewResolver)의 설정 값을 변경해야 합니다. 정확히는 ViewResolver의 경로(Path)를 수정

```
spring.mvc.view.prefix=/WEB-INF/view/
spring.mvc.view.suffix=.jsp
```

---
# 테스트

HelloController.java
```java
@Controller
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "index";
    }
}
```


view/index.jsp
```jsp
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Spring Boot Application</title>
</head>
<body>Hello, Spring Boot App</body>
</html>
```
