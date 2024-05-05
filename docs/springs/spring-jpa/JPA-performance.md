---
layout: default
title: JPA Performance
grand_parent: Springs
parent: Spring JPA
nav_order: 4
---


# jojoldu의 spring data 성능 개선

### JPA N+1 문제 및 해결방안
https://jojoldu.tistory.com/165?category=637935
### MultipleBagFetchException 발생시 해결 방법
https://jojoldu.tistory.com/457?category=637935
### JPA에서 대량의 데이터를 삭제할때 주의해야할 점
https://jojoldu.tistory.com/235?category=637935
### Querydsl 에서 OneToMany 관계에서 Left Outer Join 이 필요할 경우
https://jojoldu.tistory.com/342?category=637935
### Spring Boot Data Jpa 프로젝트에 Querydsl 적용하기
https://jojoldu.tistory.com/372?category=637935
### [Querydsl] 서브쿼리 사용하기
https://jojoldu.tistory.com/379?category=637935
### [Querydsl] 다이나믹 쿼리 사용하기
https://jojoldu.tistory.com/394?category=637935
### [Querydsl] 연관관계 없이 Join 조회하기
https://jojoldu.tistory.com/396?category=637935
### [Querydsl] Case When 사용하기
https://jojoldu.tistory.com/401?category=637935
### 더티 체킹 (Dirty Checking)이란?
https://jojoldu.tistory.com/415?category=637935
### MultipleBagFetchException 발생시 해결 방법
https://jojoldu.tistory.com/457?category=637935
### Querydsl 에서 Group by 최적화하기 (feat. MySQL)
https://jojoldu.tistory.com/477?category=637935
### JPA에서 Reader DB 사용하기 (feat. AWS Aurora)
https://jojoldu.tistory.com/515?category=637935
### Querydsl Select 필드로 Entity 사용시 주의 사항
https://jojoldu.tistory.com/518?category=637935
### JPA exists 쿼리 성능 개선
https://jojoldu.tistory.com/516?category=637935
### Querydsl (JPA) 에서 Cross Join 발생할 경우
https://jojoldu.tistory.com/533?category=637935
### JPA Entity Select에서 Update 쿼리 발생할 경우
https://jojoldu.tistory.com/536?category=637935
### JPA 사용시 @Embedded 주의사항
https://jojoldu.tistory.com/559?category=637935
### EntityQL로 OneToMany (1:N) Bulk Insert 구현하기


### 1. 페이징 성능 개선하기 - No Offset 사용하기
 * https://jojoldu.tistory.com/528?category=637935
### 2. 페이징 성능 개선하기 - 커버링 인덱스 사용하기
 * https://jojoldu.tistory.com/529
### 3-1. 페이징 성능 개선하기 - 검색 버튼 사용시 페이지 건수 고정하기
 * https://jojoldu.tistory.com/530?category=637935
### 3-2. 페이징 성능 개선하기 - 첫 페이지 조회 결과 cache 하기
 * https://jojoldu.tistory.com/531?category=637935
### Querydsl select에서 상수 사용하기
 * https://jojoldu.tistory.com/523?category=637935


### EntityQL
EntityQL 이란게 있었다고!?

 * EntityQL로 OneToMany (1:N) Bulk Insert 구현하기
   + https://jojoldu.tistory.com/560?category=637935
 * (MySQL) Auto Increment에서 TypeSafe Bulk Insert 진행하기 (feat.EntityQL, JPA)
   + https://jojoldu.tistory.com/558


# 성능팁
* [Spring Data JPA isNew 메서드](https://leegicheol.github.io/jpa/jpa-is-new/)

```java
@Transactional
public <S extends T> S save(S entity) {
    Assert.notNull(entity, "Entity must not be null.");
    if (this.entityInformation.isNew(entity)) {
        this.em.persist(entity);
        return entity;
    } else {
        return this.em.merge(entity);
    }
}
```
=> DB에서 먼저 조회해서 값이 있는지 체크하게 된다.


merge는 일반적인 update가 아닌, 데이터 자체를 덮어씌우는 update를 수행한다. 그래서 수정하지 않는 값도 변경처리되는 경우가 생기게 된다.

JPA의 Auditing 기능을 사용하면 isNew 메서드를 쉽게 구현할 수 있다.

Auditing은 @CreatedDate, @CreatedBy, @LastModifiedBy, @LastModifiedDate
네 개의 Annotation을 지원한다.

isNew 메서드 호출 시점에 @CreatedDate가 아직 null이라면 DB에 insert가 되지 않은 시점이라고 판단할 수 있게 된다.

```java
@Data
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Users implements Persistable<String> {

    @Id
    private String id;

    private String name;

    private int age;

    @CreatedDate
    private LocalDateTime createDate;


    @Override
    public String getId() {
        return id;
    }

    @Override
    public boolean isNew() {
        return createDate == null;
    }

}
```


---
# 우아한형제들

### HikariCP Dead lock에서 벗어나기

https://techblog.woowahan.com/2663/
https://techblog.woowahan.com/2664/

=> Entity의 Primary Key 관련해서 시퀀스 관련한 내용..
