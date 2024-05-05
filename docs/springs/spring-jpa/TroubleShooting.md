---
layout: default
title: Trouble Shooting
grand_parent: Springs
parent: Spring JPA
nav_order: 4
---


# Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'JPQLMemberRepository' defined in file [/Repository/JPQLMemberRepository.class]: Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'javax.persistence.EntityManager' available: expected single matching bean but found 2: getEm,org.springframework.orm.jpa.SharedEntityManagerCreator#0


### case1. https://tjdwns4537.tistory.com/m/67

해결 방법
1. IoC컨테이너에서 EntityManagerFactory, EntityManager를 따로 빈 등록하는 것이 아닌 하나의 함수에 묶어서 빈 등록
2. 레포지토리에서는 엔티티 매니저만 주입을 받음
3. 테스트코드에서는 레포지토리만 주입 받음

이렇게 수정하는게 코드의 영향범위를 줄이기 때문에(SOLID) 더 나은 방법으로 보인다.

### case2. https://programmer-chocho.tistory.com/78


### 더 알아보기-1
[Entity, EntityManager and Persistence Context](https://velog.io/@koo8624/Spring-EntityManager)



참고로 아래 그림은 Entity Manager를 Constructor Injection으로 세팅하는 경우. Autowired 어노테이션은 생략가능하다.
![](/images/spring-jpa/entitymanager_constructor_injection.png)

=> "multi datasource entity manager constructor injection" <= 이 키워드로 검색해보자

### 더 알아보기-2
https://www.inflearn.com/questions/252369/bean-%EB%93%B1%EB%A1%9D%EC%8B%9C-entitymanager-%EC%98%A4%EB%A5%98-%EC%A7%88%EB%AC%B8%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4
> 주입을 하려면 스프링으로 관리하는 Bean이 되어야겠죠?
> JPA와 스프링은 각자의 방식(영속성 컨텍스트와 스프링 컨테이너)으로 빈을 관리하거나 엔티티를 관리합니다.


https://stackoverflow.com/questions/22301426/no-qualifying-bean-of-type-javax-persistence-entitymanager



# Can't set JPA naming strategy after configuring multiple data sources (Spring 1.4.1 / Hibernate 5.x)
https://stackoverflow.com/questions/40509395/cant-set-jpa-naming-strategy-after-configuring-multiple-data-sources-spring-1

stack over flow 에서 나온 내용으로는 소스코드를 변경하거나, application.properties를 수정해서 해결하는 듯하다.


# 어느날 부터인가 컬럼명이 카멜케이스로 나오고 있다. 왜이러지?

Springboot 특정 버전 이상부터 프로퍼티 설정이 변경되었다.



프로젝트에 대소문자 구별이 있으면 추후에도 문제가 생길 여지가 있어 모두 소문자로 사용하기 위한 전략을 택했다.



ImprovedNamingStrategy 이용시 카멜 케이스를 언더스코어 버전으로 변경해준다.

Spring Boot 디폴트는 org.springframework.boot.orm.jpa.SpringNamingStrategy이다.

SpringNamingStrategy는 ImprovedNamingStrategy 을 상속한 클레스이다.

* Hibernate 4
```
spring.jpa.hibernate.naming.strategy=org.hibernate.cfg.ImprovedNamingStrategy
```

* Hibernate 5
```
# 논리(설명)
spring.jpa.hibernate.naming.implicit-strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy


# 물리(영문, 테이블명)
spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
```


* java configuration 버전
```
hibernateProperties.setProperty("hibernate.physical_naming_strategy" , "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");
hibernateProperties.setProperty("hibernate.implicit_naming_strategy" , "org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy");
```

### Configure Hibernate Naming Strategy

Hibernate는 두 개의 다른 명명 전략 을 사용 하여 객체 모델에서 이름을 상응하는 데이터베이스 이름으로 매핑한다. 물리적 구현과 암시 적 전략 구현의 정규화 된 클래스 이름은 각각 spring.jpa.hibernate.naming.physical-strategyand spring.jpa.hibernate.naming.implicit-strategy속성 을 설정하여 구성 할 수 있습니다 . 또는 응용 프로그램 컨텍스트에서 ImplicitNamingStrategy또는 PhysicalNamingStrategyBean을 사용할 수있는 경우 Hibernate는이를 사용하도록 자동으로 구성됩니다.

기본적으로 스프링 부트는 물리적 이름 지정 전략을 사용하여 구성합니다 SpringPhysicalNamingStrategy. 이 구현은 Hibernate 4와 같은 테이블 구조를 제공한다 : 모든 도트는 밑줄로 대체되고 낙타의 대문자는 밑줄로 대체된다. 기본적으로 모든 테이블 이름은 소문자로 생성되지만 스키마에 필요하면 해당 플래그를 무시할 수 있습니다.

예를 들어 TelephoneNumber엔티티가 telephone_number테이블에 매핑됩니다 .

만약 당신이 Hibernate 5의 디폴트를 대신 사용하고 싶다면, 다음 프로퍼티를 설정한다 :


기본 변수이름을 그대로 이용한다.
```
spring.jpa.hibernate.naming.physical-strategy = org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

전략 선택
```
SpringPhysicalNamingStrategy(Default)  =>  camel case를 underscore 형태로 변경
PhysicalNamingStrategyStandardImpl     =>  변수 이름을 그대로 사용
```

# 'Many To One' attribute type should not be 'Mapped Superclass'
Mapped Superclass는 엔티티가 아니기 때문에, 관계의 대상이 될 수 없다.
> JPA does support polymorphic associations but the target classes have to be part of an inheritance hierarchy

* 관련 리소스
  + https://stackoverflow.com/questions/2928916/hibernate-persisting-polymorphic-joins/2929796#2929796
