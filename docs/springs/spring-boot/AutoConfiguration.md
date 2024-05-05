---
layout: default
title: Spring Boot Auto Configuration
grand_parent: Springs
parent: Spring Boot
nav_order: 4
---

Bean을 자동으로 구성해주는 기능.

# 관련 중요 어노테이션

* @SpringBootApplication
  + @Configuration, @EnalbeAutoConfiguration, @ComponentScan 이 결합된 Annotation
  + ApplicationContext를 구성하는데 필요한 메타데이터 제공
* @EnableAutoConfiguration
* @Import &  @ImportSelector (조건)
  + Import할 하나 이상의 대상 지정하고,
  + ImportSelector를 이용해서 등록할 대상을 동적으로 구성 가능
* AutoConfigurationImportSelector
  + 어플리케이션의 환경에 맞춰서 Auto Configuration의 후보들을 선택할 수 있게 한다.
* AutoConfiguration.imports

* @AutoConfiguration
  + AutoConfiguration Class 명시 & 순서 제어
  + 스프링에서 Auto Configuration으로 처리할 수 있는 후보를 명시하는 역활
  + before, after 속성을 통해 Auto Configuration 시 처리 순서를 제어할 수 있음.
* Condition & @Conditional
  + 조건에 따라서 적용할지 안할지
* @ConditionalOn*
  + 자주쓰는 Condition 구현체들을 미리 @Conditional(구현체)로 미리 구현해 제공
  + 20개 정도가 미리 작성되서 손쉽게 사용할 수 있다.
* @ConfigurationProperties
  + 외부 설정(외부 파일 등)을 자바 객체에 Binding하기 위한 Annotation
* @EnableConfigurationProperties
  + Binding한 자바 객체 주입
