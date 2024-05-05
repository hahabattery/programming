---
layout: default
title: Spring Framework VS Spring Boot
parent: Springs
nav_order: 22
---

Spring Framework와 Spring Boot 와의 차이점을 정리한다.

Spring Boot는 Spring을 좀 더 간단하게 설정해서 사용하거나 stand-alone으로 구동하기 위한 모듈이 추가적으로 들어가 있는 것. 그래서 크게 다르지는 않는다.



* "Spring Framework VS Spring Boot Framework"


# Spring boot를 언제 써야할까 - zepinos
[Spring VS Spring Boot 차이점](https://programforlife.tistory.com/68) <= 원본이 어딘지는 모르겠지만, 잘 정리되어있다.

[프롤로그](https://zepinos.tistory.com/84)

[spring boot는 어떤 구조일까](https://zepinos.tistory.com/85)

[외장 WAS가 반드시 필요한가](https://zepinos.tistory.com/86)

[외장 WAS만이 할 수 있는 것](https://zepinos.tistory.com/87)


# AutoConfiguration

spring-boot-autoconfigure-X.X.X.jar 의 spring.factories 파일에 자주 사용되는 라이브러리들의 자동설정파일이 있음.

이 설정들은 @EnableAutoConfiguration이 true로 활성화 될때 동작함(@SpringBootApplication 에 설정되어 있음).

@EnableAutoConfiguration 어노테이션에서 AutoConfigurationImportSelector 클래스를 import하는데, AutoConfigurationImportSelector 클래스는 자동설정할 후보빈을 불러오고 제외하거나 중복된 설정들을 제외하는 등의 작업을 거친 후 자동 설정할 빈을 결정한다.

실행 옵션에서 "--debug" 를 넣으면 auto configuration 과정을 볼 수 있음.


추가로 IntelliJ에서는 별도의 설정 없이 Actuator의 기능을 제공해서, 어떤 Bean들이 등록됐는지 다음과 같이 쉽게 확인 할 수 있음.

![](./imgs/springboot-autoconfiguration-01.png)



## 빈의 등록과 자동 설정에 필요한 파일
 * META-INF/spring.factories
   + 자동 설정 타깃 클래스 목록
   + 이곳에 선언된 클래스들이 @EnableConfiguration 사용 시 자동 설정 타깃이 된다.

 * META-INF/spring-configuration-matadata.json
   + 자동 설정에 사용할 프로퍼티 정의 파일
   + 미리 구현되어 있는 자동 설정에 프로퍼티만 주입시켜주면 된다.
   + 변경을 위해서는 application.properties 나 application.yml에 프로퍼티 값을 추가한다.

 * org/springframework/boot/autoconfigure
   + 미리 구현해놓은 자동 설정 리스트
   + 모두 자바 설정 방식을 따른다.


## FlywayAutoConfiguration 의 auto configuration 분석


```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(Flyway.class)
@Conditional(FlywayDataSourceCondition.class)
@ConditionalOnProperty(prefix = "spring.flyway", name = "enabled", matchIfMissing = true)
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, JdbcTemplateAutoConfiguration.class,
		HibernateJpaAutoConfiguration.class })
@Import({ FlywayEntityManagerFactoryDependsOnPostProcessor.class, FlywayJdbcOperationsDependsOnPostProcessor.class,
		FlywayNamedParameterJdbcOperationsDependencyConfiguration.class })
public class FlywayAutoConfiguration {
```

#### 1. Configuration(proxyBeanMethods = false)
대부분의 AutoConfiguration 설정의 경우 proxyBeanMethods = false로 설정되어 있다. 기본 설정은 proxyBeanMethods = true 인데, CGLIB을 통해 런타임에 동적으로 Configuration 객체의 프록시를 생성해준다. 필자가 이해한 바로는 아래의 설정을 진행하는 데 필요하다.



```java
@Configuration
public class Config {

  @Bean
  SomeBean someBean() {
    return new SomeBean();
  }

  @Bean
  NothingBean nothingBean() {
    return new NothingBean(someBean());
  }
}
```

@Configuration에서는 메소드 호출로 내부에서 정의한 Bean을 바로 사용할 수 있다. 기본 설정을 사용하면 Configuration의 Proxy가 생성되고, Proxy는 그 내부의 Bean이 한 번만 생성되도록 관리한다. 즉, 기본 설정에서는 SomeBean은 한 번만 생성되고, proxyBeanMethod = false를 사용하면, NothingBean을 생성하는 시점에서 또 다른 SomeBean이 생성되어 총 두 번 생성이 된다. 하지만, 이런 방식은 모든 Configuration에서는 필수적이지 않으며, Proxy를 생성하는 부분에서 성능적인 이슈가 있다고 한다. 자세한 부분은 다음 SpringBoot 이슈 를 참고하길 바란다.

[Use @Configuration(proxyBeanMethods=false) wherever possible](https://github.com/spring-projects/spring-boot/issues/9068)





#### 2. @Conditional-#

Spring4부터 제공하는 Conditional 어노테이션은, AutoConfiguration의 구성에 자주 사용된다. 조건에 따라 해당 자동 설정이 적용 여부를 결정한다. 대표적으로 몇 가지를 적으면 다음과 같다.

 * @ConditionalOnClass: 해당 Class가 classpath에 존재할 때 조건을 만족한다. 위의 경우 Flyway에 대한 dependency를 추가해주지 않았기 때문에, 빨간색으로 표시가 되었고, FlywayAutoConfiguration이 동작하지 않게 된다.
 * @ConditionalOnMissingBean: 특정 Bean이 사전에 정의되지 않을 때 조건을 만족한다.
 * @ConditionalOnProperty: 특정 property를 명시해주고 일치할 때 조건을 만족한다.
 * @AutoConfigurationBefore / @AutoConfigurationAfter

AutoConfiguration의 등록 순서를 결정한다. FlywayAutoConfiguration 같은 경우에는 DatasourceAutoConfiguration, JdbcTemplateAutoConfiguraion, HibernateJpaAutoConfiguration 이후에 설정이 진행된다.

참고) [스프링 부트의 Autoconfiguration 원리 및 만들어 보기](https://donghyeon.dev/spring/2020/08/01/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%EC%9D%98-AutoConfiguration%EC%9D%98-%EC%9B%90%EB%A6%AC-%EB%B0%8F-%EB%A7%8C%EB%93%A4%EC%96%B4-%EB%B3%B4%EA%B8%B0/)
