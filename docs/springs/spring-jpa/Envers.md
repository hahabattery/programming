---
layout: default
title: Envers
grand_parent: Springs
parent: Spring JPA
nav_order: 4
---

hibernate 에서 만든 데이터 변경 이력을 로깅하기 위한 라이브러리

# 세팅

### 의존성
의존성 추가
2020.4월 기준 recent stable version인 hibernate-envers 라이브러리를 build.gradle에 추가해줍니다.

```
compile group: 'org.hibernate', name: 'hibernate-envers', version: '5.4.14.Final'
```

### 변경이력 저장할 테이블 설정

@Audited로 변경이력 저장할 테이블 생성
변경이력을 추적할 엔티티에 @Audited 어노테이션을 추가해줍니다.
만약 extends 한 엔티티의 이력도 추가되길 원한다면 @AuditOverride(forClass=BaseEntity.class) 도 추가합니다.
그러면 기본적으로 총 두개의 테이블이 생성되게 됩니다.
예를들어 customer 엔티티의 상단에 @Audited를 추가하면 다음과 같은 customer_aud, revinfo 테이블이 생성됩니다.
```java
@Entity
@Audited
public class Customer {... }
```

### 변경이력이 저장되는 테이블 설정
아래와 같이 RevisionEntity를 설정하지 않으면 테이블 명이 "revinfo" 로 설정된다.
```java
@Entity
@RevisionEntity
@Table
public class RevisionDomain implements Serializable {
  ...
}
```
