---
layout: default
title: Spring Boot Overview
grand_parent: Springs
parent: Spring Boot
nav_order: 1
---


 * 디렉토리이름만 보고도 내용을 알 수 있도록 디렉토리명을 변경하자.

 * SpringBoot 의 버전 정보가 필요하다면!
   + https://mvnrepository.com/artifact/org.springframework.boot/spring-boot


---
# Spring Profiles

 * spring profile 관련 참고할만한 링크
   * https://www.baeldung.com/properties-with-spring
   * https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties


 * maven profile과 Spring Profile
   * Maven Profile은 환경마다 빌드를 새로 해야 하는 문제가 있으니 이것보다는 spring profile 이 추천됨
   * Maven Profile을 이용한 예제 소스
     * LMS-Marv/src/daekyo_lms

### 프로파일 설정 방법
1. 운영체제 환경 변수에 실행할 프로파일을 설정
```
  $ vi $HOME/.bashrc
  export SPRING_PROFILES_ACTIVE=dev

  $ $HOME/.bashrc
```

2. JVM 옵션으로 실행할 프로파일을 전달
```
  $ java -Dspring.profiles.active=dev -jar some-project.jar
```



### 여러개의 profile이 설정하는 경우

 * 프로파일이 여러개 설정된 경우에는 IDE에서 VM options으로 -Dspring.profiles.active=test 로 지정가능함.
 * 실제 환경에서 실행시에는 아래처럼 옵션으로 설정이 가능하다.
 ```
 java -jar -Dspring.profiles.active=default  ./kk-0.0.1-SNAPSHOT.jar
 ```



 * profiles를 설정해주지 않으면 기본 "default" 라는 이름으로 동작한다.

 * 다음은 문서에 명시된 profile이 지정되지 않았을 때의 action이다.
   * if no profiles are explicitly activated, then properties from application-default.properties are loaded




##### 여러개의 파일을 사용하는 경우
 ```
 application-{profile}.properties 형식으로 다음과 여러개의 파일을 생성하면 된다.
 application-default.properties
 application-dev.properties
 application-prod.properties
 ```

##### 하나의 파일에 여러개의 profile 설정을 하는 경우(yaml)

 * 한 파일에 여러 profile설정이 가능하다.(yaml파일을 설정 파일로 이용할 경우 @PropertySource애너테이션을 통해 값을 읽어 올 수 없다는 점이 단점이다.)
 * 다음은 한 파일에 ---로 구분해서 profile을 설정한 예
```
 spring:
  profiles:
    active: default
logging:
  level:
    root: debug
---
spring:
  profiles: dev
logging:
  level:
    root: info
---
spring:
  profiles: test
logging:
  level:
    root: debug
  level.org.hibernate:
    SQL: debug
    type.descriptor.sql.BasicBinder: trace
 ```


### application.properties 파일에 대해서

 * application.properties 파일은 가장 기초적인 프로파일 설정이고, profile별로 다른 설정을 application-{profile명}.properties 형식으로 설정할 수 있다.
 * profile별의 설정은 application.properties의 설정을 overriding한다.
 * 만약에 application.properties 파일이 없는 경우에는 application-default.properties 파일이 로딩되었다.

> In addition to application.properties files, profile specific properties can also be defined using the naming convention application-{profile}.properties.
> Profile specific properties are loaded from the same locations as standard application.properties, with profiles specific files overriding the default ones.


### 위의 profile설정보다 더 커스터마이징하는 경우의 예

 * https://velog.io/@kingcjy/%ED%99%98%EA%B2%BD%EC%97%90-%EB%A7%9E%EB%8A%94-Spring-Profile-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0


### 설정에 대한 여러가지 access방법
application.properties에 접근하는 방법은 여러가지 있는데. 아래처럼 Environment를 이용해서 접근도 가능하다.
```
@Autowired
private Environment env;

env.getProperty("application.properties에 설정한 프로퍼티명");
```

### 테스트 환경

##### InteliJ에서는
 * test 디렉토리에 resources 디렉토리를 만들고, 안에 application.yml 파일을 위치시키면 동작함
```
spring:
  profiles:
    active: test
```

 * 테스트 케이스 실행의 경우에는 , VM Option(IDE의 설정이나 -Dspring.profiles.active=local) 을 지정하여도 적용되지 않았다.
 * 테스트시에 profile별로 설정을 나누기 위해서는application.properties나 application-default.properties 파일에 spring.profile.active 설정을 해야한다.
 * 테스트에 프로파일을 나누는 것이 효과적이지 않고, @TestPropertySource 를 이용하는 것이 편리한거 같다.


 * 이클립스에서 위 방법으로는 안된다고 한다(확인 X)


 * test/resources 폴더에 파일이 존재하면, 우선적으로 적용됨.
 * test/resources 폴더에 설정 파일이 존재하지 않으면 main/resources 폴더의 파일이 적용되었음.
   * 이 동작은 아마 @SpringBootTest 를 이용하는 경우에는 전체 컨테이너를 적용하기 때문인거 같음.
   * @SpringBootTest 이용시에는 VMOption은 먹지 않고, application.properties 파일의 설정이 적용되었다. 테스트시에 VMOption을 설정하는 거 부터가 의미 없기 때문에 이게 당연한 듯 하다.
   * 결론) 만약에 테스트케이스 작성시에 test/resources/ 폴더에 설정파일을 생성하지 않고 SpringBootTest를 이용하고자 한다면, main/resources/application.properties파일에는 테스트를 위한 profile로 설정되어야 한다.


---
# Multi DataSource 이용 방법
"Springboot Multi Datasource" <- 이걸로 검색
