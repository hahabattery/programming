---
layout: default
title: JPA Overview
grand_parent: Springs
parent: Spring JPA
nav_order: 1
---


# 리소스
 * [Jakarta Persistence](https://jakarta.ee/specifications/persistence/3.0/jakarta-persistence-spec-3.0.html)

 * [Spring-Jpa Best Practices](https://github.com/cheese10yun/spring-jpa-best-practices)


# JPA의 특징
 * JPA는 동시성에 대한 것은 모두 데이터베이스의 트랜잭션 처리에 위임함.


# JPA vs Hibernate
Hibernate는 JPA의 구현체중의 하나이다. Hibernate의 동작의 특성을 정리한다.


JPA는 기술 명세이다. JPA에 대해서 정리한다.

 * JPA 는 인터페이스
 * hibernate는 구현체  
   + https://hibernate.org/orm/releases/ <= hibernate의 릴리즈 노트 (구현한 JPA 버전정보를 확인할 수 있다)
 
### 관련 패키지

 * javax.persistence 로 시작하는 패키지는 JPA의 스펙에 해당하는 인터페이스


# mapping between Java type and database types
 * [How to persist LocalDate and LocalDateTime with JPA 2.1](https://thorben-janssen.com/persist-localdate-localdatetime-jpa/)
   + This article explains about relationship among Java, JPA, Hibernate.

The mapping between Java type and database types can be influenced by the specific JPA implemetation (like hibernate) and database.

JPA handles the conversion between the Java type and the MySQL type. When reading from the database, it converts the DATETIME value to Timestamp or LocalDateTime. When writing to the database, it converts the Timestamp or LocalDateTime value to DATETIME.  This automatic conversion works for many common Java types. For more complex types, or for custom conversions, you can use the @Convert annotation to specify a custom AttributeConverter class that handles the conversion.


# Hibernate change orders of query execution.
https://stackoverflow.com/questions/18853146/insert-after-delete-same-transaction-in-spring-data-jpa

When flushing, Hibernate executes all inserts before delete statements.


# 네이밍 룰
https://www.baeldung.com/hibernate-naming-strategy



# 어플리케이션의 DB계정 권한
 * hibernate.hbm2ddl.auto 옵션을 운영서버에서는 사용하지 않는 것이 좋고,
 * 더 좋은 것은 어플리케이션의 DB계정 권한에서 Create / Drop / Alter TABLE 를 주지 않는 것이 좋다.  


# application.yml 설정관련 노트
 * jpa.hibernate.ddl-auto : create
   * 어플리케이션 실행시점에 테이블을 모두 지우고 생성한다는 의미
   * create-drop은 drop table 쿼리를 실행해서 삭제까지 처리함.
 * jpa.properties.hibernate.show_sql
   * SQL을 콘솔에 출력
 * logging.level.org.hibernate.SQL
   * SQL을 로깅파일에 출력

 * jpa.properties.hibernate.dialect: org.hibernate.dialect.Oracle12cDialect
   * JPA는 dialect (방언)기반으로 동작하기 때문에,
   * dialect설정을 바꾸면 쿼리가 해당 데이터베이스에 맞춰서 변경된다.

 * hibernate.default_batch_fetch_size: 1000
   * ToMany관계인 경우에는 @BatchSize(size = 1000)를 Entity 클래스의 멤버에 적용하면 된다.
   * ToOne관계인 경우에는 @BatchSize(size = 1000)를 Entity 클래스에 적용하면 된다.
   * fetch join하지 않아서 LAZY 초기화(Proxy초기화)되는 경우에 적용받음.

 * org.hibernate.type: trace
   * query parameter가 파라미터별 정보가 로그에 남는다.



# 어노테이션 관련 설명

 * @Repository
   * 따라가보면 @Component 어노테이션 적용되어있어서,
   * Component Scan의 대상이므로, 자동으로 Spring Bean에 등록이 됨.
     + 이건 @Service도 동일함.
 * @Repository 어노테이션의 기능
   * DAO 개념으로 엔티티를 조회하기 위해서 DB에 접근하는 객체
   * Component scan의 대상이 되도록 함
   * 다른 어노테이션과 다른 점은 JPA의 예외를 Spring에서 공통적으로 처리할 수 있는 예외로 변경하는 처리를 해준다.

 * @PersistenceContext
   * Spring Boot에서는 모든 것이 Spring Container위에서 동작하게 되는데,
   * Spring Container가 PersistenceContext가 붙어있으면 자동으로 EntityManager를 주입해준다.

 * @Test @Transactional @Rollback(false)
   * Transactional 어노테이션에 @Test가 있으면, 테스트후에 롤백이 일어난다.
   * 이러한 동작은 데이터가 남는 경우, 반복적인 테스트를 하지 못하기 때문이라고 한다.
   * 롤백을 하지 않게 하기 위해서는 @Rollback(false) 를 추가하면 된다.
   * Spring이 롤백을 하면, DDL 쿼리를 실행할 필요조차 없기 때문에, 쿼리로 확인하기위해서는 @Rollback(false)를 추가해줘야 한다.



 * @AllArgsConstructor
   * Lombok관련. 모든 멤버변수가 포함된 생성자 생성
 * @RequiredArgsConstructor
   * Lombok관련. final이 있는 멤버변수에 대해서만 생성자 생성.

 * @Data
   * Getter, Setter, RequiredArgsConstructor, ToString, ...등이 포함된 어노테이션.

 * @RequestBody
   * JSON데이터를 객체로 매핑시켜줌

 * @RestContoller
   * @Controller + @ResponseBody

 * @JsonIgnore
   * jackson library로 json생성시에 제외하는 설정. 필드에 지정.
   * entity 클래스에 넣는 것은 presentation 계층에 대한 내용이 들어오게되므로 비추.

---
# data projection

 * The best way to map a projection query to a DTO (Data Transfer Object) with JPA and Hibernate
   + https://vladmihalcea.com/the-best-way-to-map-a-projection-query-to-a-dto-with-jpa-and-hibernate/



---
# java8 time

 * Java 8 Date(Time) 와 JPA 그리고 스프링 부트
   + https://www.popit.kr/java-8-datetime-%EC%99%80-jpahiberante-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8/
   + 하이버네이트 5.2부터 hibernate-java8 의존성 추가 필요 없이 Java 8 날짜와 시간을 사용가능하다.



---
# 팁

 * 일대다 관계에서 엔티티 클래스에 List를 넣는데, 아래처럼 변수 선언시점에 초기화해주면, null exception으로부터 안전해진다.
 ```
 private List<Member> members = new ArrayList<>();
 ```

### equals()와 hashCode()
 * equals() 재정의시에 hashCode()도 재정의해줘야, 해시를 사용하는 hash map같은 콜렉션에서 효율적으로 사용할 수 있다.

### 어노테이션 사용법
 * Ctrl누르고 어노테이션을 클릭해서 정의를 볼 수 있는데, 여기에 보통 사용법이 잘 나와있다.

### 양방향 연관관계시
 * toString()이용시에 양방향으로 toString()이 호출되면서 무한루프에 빠질 수 있다.
 * lombok 에서 toString 자동 생성 옵션은 끄자.
 * JSON 생성 라이브러리 이용시에 문제가 될 수 있다(controller에서 Entity클래스를 반환하는 경우)

##### jackson library에서 무한루프에 빠지는 이슈를 방지하기 위해서는
 * 양방향 연관계에서 한쪽에 @JsonIgnore 를 넣어줘야 한다.


##### JPQL 함수
 * DB에 종속적이지 않은 JPQL 표준함수같은 경우는 몇개 없는데...
 * Ctrl+N으로 Class를 Mysql로 검색해보면, 해당 DB에 종속적인 함수가 미리 등록되있는 것을 알 수 있다.


### 제약조건

 * 모든 테이블에 PK는 generated되는 값으로 쓰는 것이 추천된다.
   * PK가 비즈니스적으로 의미없게 되지만, 시스템의 유연성이 좋아진다.

### Dirty Checking (변경감지)
 * 변경감지는 아마 레코드마다 따로 업데이트가 될 것이기 때문에, 벌크 처리를 하도록 하는 것이 성능상 좋다.

### Entity 캐슁

 * Entity 를 바로 캐시로 관리하면, 꼬일 수 있기 때문에, DTO를 캐시하자.
 * 하이버네이트 2차 캐시라는게 있지만, 실무에서 적용하기는 까다롭다고 한다.

### Inheritance 설정
 * 데이터가 하루에 100만건씩 쌓이거나 하는 상황에서는 테이블 구조를 심플하게 가져가는 것이 중요하기 때문에,
 * 상속관계를 사용하지 않고, JSON으로 하위 클래스를 넣어주는 등의 변경이 필요 할 수 있다.

### Cascade
```
cascade = CascadeType.ALL
```

 * cascade설정은 부모 하나(이것을 owner가 하나라고 한다)에서 관리하는 경우에 사용된다.
 * 하지만 부모가 여러개가 되는 경우에는 사용하면 안된다.

 * 아래 2가지 조건이 맞는 경우에만 사용해야 한다.
   * 부모와 자식간의 라이프사이클이 유사할 떄
   * 단일 소유자가 하나일 때(부모가 하나일 떄)

### EntityGraph

 * 간단하면 EntityGraph를 쓸 수 있고,
 * 복잡해지면, JPQL의 fetch join을 활용하는 것이 낫다.



# 일반 join과 fetch join의 차이

[일반 Join과 Fetch Join의 차이](https://cobbybb.tistory.com/18)


### 일반 Join
Fetch Join과 달리 연관 Entity에 Join을 걸어도 실제 쿼리에서 SELECT 하는 Entity는
오직 JPQL에서 조회하는 주체가 되는 Entity만 조회하여 영속화
조회의 주체가 되는 Entity만 SELECT 해서 영속화하기 때문에 데이터는 필요하지 않지만 연관 Entity가 검색조건에는 필요한 경우에 주로 사용됨

일반 join은 실제 쿼리에 join을 걸어주기는 하지만 join대상에 대한 영속성까지는 관여하지 않기 때문에, 추후에 접근시에 에러가 발생할 수 있다.

또한, 일반 Join을 적절히 이용하여 필요한 Entity만 영속성 컨텍스트에 올려서 사용하는 것이 괜한 오작동을 미리 방지할 수 있다.



### Fetch Join
SQL 조인 종류가 아닌, JPQL에서 성능 최적화를 위해 제공하는 기능이다. 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회하는 기능.

조회의 주체가 되는 Entity 이외에 Fetch Join이 걸린 연관 Entity도 함께 SELECT 하여 모두 영속화
Fetch Join이 걸린 Entity 모두 영속화하기 때문에 FetchType이 Lazy인 Entity를 참조하더라도
이미 영속성 컨텍스트에 들어있기 때문에 따로 쿼리가 실행되지 않은 채로 N+1문제가 해결됨


# Fetch Join 시의 주의점

[페치 조인(Fetch Join)의 일관성 - 대상에 별칭 사용하면 안되는 이유](https://loosie.tistory.com/750)

https://www.inflearn.com/questions/15876

https://stackoverflow.com/questions/17431312/what-is-the-difference-between-join-and-join-fetch-when-using-jpa-and-hibernate

https://docs.jboss.org/hibernate/orm/3.3/reference/en/html/queryhql.html#queryhql-joins



---
# Lazy

 * 연관관계 설정에 LAZY가 설정된 경우, JPA는 바로 해당 엔티티를 DB에서 가져오지 않고
 * PROXY관련 클래스인 ByteBuddyInterceptor()를 대신 설정한다.


---
# JPQL
 * DB의 distinct는 row가 모두 값아서 중복으로 판단하는데,
 * JPQL은 DISTINCT가 걸려있으면, DB에 ditinct 키워드를 넣어서 실행하고,
 * 루트 엔티티(Order)의 객체의 PK가 동일하면, 실행 결과를 리턴할 떄, 중복된 것으로 판단해서 제거하고 리턴한다.

### JPA JPQL 서브쿼리의 한계

 * from절의 서브쿼리(인라인 뷰라고 함)는 지원하지 않음.
 * DB가 지원해주는 많은 기능을 이용해서, 복잡한 UI를 위한 정보를 만들다 보면 from절에 서브쿼리로 들어가게 된다.
 * DB로부터는 필터링(where),그룹핑(group by)해서 데이터를 가져오는데 집중하고, 어플리케이션 레벨에서 로직을 처리하고, UI레벨에서 이쁘게 표현하는 것은 UI어플리케이션(프레젠테이션)에게 맡기는 식으로 처리하는 것이 권장된다.

 * 백오피스와 같은 성능이 중요하지 않은 경우는 복잡한 쿼리를 의도적으로 만드는 경우가 있는데, 이런 경우는 차라리 쿼리를 2,3번 나누는 것이 나을 수도 있다.
   * SQL은 집합적으로 아이디어로 로직을 작성해야되는데, 어플리케이션 로직은 시퀀셜하게 처리해서 풀어낼 수 있기 때문에
   * SQL AntiPatterns 라는 책에서 나오는 내용.

### 성능
 * 묵시적 inner join이 발생하면 직관적이지 않기 때문에 추후에 성능 튜닝이 어려워지는 문제가 생긴다.
   * 따라서 최대한 JPQL과 실제 실행되는 SQL을 비슷하게 맞춰주는게 좋다.

##### @QueryHint의 readOnly 와 @Transaction의 readOnly 차이

 * @QueryHint의 readOnly는 스냅샷을 만들지 않기 때문에, 메모리가 절약되고, 스냅샷과의 비교(dirty checking)를 하지 않게됨.
 * @Transaction(readOnly=true)는 트랜잭션 커밋 시점에 flush를 하지 않기 때문에 이로 인한 dirty checking 비용이 들지 않습니다. 따라서 cpu가 절약됨.
 * 따라서 읽기 전용 데이터는 @Transaction(readOnly=true)로 설정하시면, @QueryHint의 readOnly까지 적용됨.


---
# Transaction and Lock

 * JPA hibernate 책의 맨 뒤편에 해당 내용이 있다고 한다.

 * lock에 대한 코멘트
```
락은 정말 데이터를 동시에 변경하면 안되는 이슈가 있을 때만 보수적으로 사용해야 합니다.
추가로 실무에서 락은 최후의 보루 정도로 생각해야 합니다.
생각해보면 우리 일상의 거의 모든 곳에 락을 걸어야 하는데, 그렇지 않아도 서비스는 잘 동작합니다.
왜냐하면, 정말로 같은 시간에 같은 데이터를 변경하는 일은 거의 없기 때문입니다.
비즈니스상 크리티컬 하더라도 정말 동시에 같은 데이터를 수정하는 일을 매우 드뭅니다.
제 경험상 락을 풀어두어서 발생하는 이슈보다 락을 걸어서 발생하는 이슈가 더 많았습니다.
특히 데이터베이스에 따라서 동작하는 방식도 달라서 꼭 실무에서는 확인하고 검증해본 다음에 사용해야 합니다.
```

### Lock Modes
 * 참조)
 * https://www.baeldung.com/jpa-pessimistic-locking
 * https://www.baeldung.com/jpa-optimistic-locking
   + => 이걸 참조해서 테스트 케이스를 만들 수 있다.

JPA specification defines three pessimistic lock modes that we're going to discuss:

 * PESSIMISTIC_READ
   + allows us to obtain a shared lock and prevent the data from being updated or deleted.
 * PESSIMISTIC_WRITE
   + allows us to obtain an exclusive lock and prevent the data from being read, updated or deleted.
 * PESSIMISTIC_FORCE_INCREMENT
   + works like PESSIMISTIC_WRITE, and it additionally increments a version attribute of a versioned entity.

 * PersistenceException
   + we can obtain only one lock at a time. If it's impossible, a PersistenceException is thrown.

##### Exceptions

 * PessimisticLockException
   + indicates that obtaining a lock or converting a shared to exclusive lock fails and results in a transaction-level rollback.
 * LockTimeoutException
   + indicates that obtaining a lock or converting a shared lock to exclusive times out and results in a statement-level rollback.
 * PersistenceException
   + indicates that a persistence problem occurred. PersistenceException and its subtypes, except NoResultException, NonUniqueResultException, LockTimeoutException and QueryTimeoutException, mark the active transaction to be rolled back.


### Lock Scope

 * PessimisticLockScope.NORMAL
   + default
   + With this locking scope, we lock the entity itself. When used with joined inheritance, it also locks the ancestors.
 * PessimisticLockScope.EXTENDED
   + it's able to block related entities in a join table.Simply put, it works with entities annotated with @ElementCollection or @OneToOne, @OneToMany, etc. with @JoinTable.



---
# 영속성
영속성 관리에 대해서 동작이 헷갈리기 쉽니다.

Examples/SpringJpa/PDF/자바 ORM 표준 JPA 프로그래밍 - 기본편/03.영속성관리.pdf 파일을 참조해보자.

### 영속성 컨텍스트 초기화

 * 아래 코드로 변경사항을 반영하고, 영속성 컨텍스트를 초기화해서
 * 1차 캐쉬에 데이터가 없어지기 때문에, DB에 쿼리가 실행되는 것을 확인할 수 있다.
```
em.flush()
em.clear()
```

### 영속성 컨텍스트관련

 * https://doublesprogramming.tistory.com/259



---
# H2

 * application.properties나 application.yml 파일에 spring.datasource 설정이 없으면, 내부 H2로 동작함.

### 외부 H2 이용시에 데이터베이스 생성 방법

 * H2 데이터베이스를 다운로드 해서 설치
 * 웹 콘솔 접속
   * window에서는 설치 경로/bin 디렉토리에서 h2.bat 파일 실행
 * 데이터베이스 파일 생성
   * 웹 콘솔의 JDBC URL입력란에 jdbc:h2:~/db명  <= 이렇게 입력하면 db명.mv.db 파일이 생성된다.
 * 데이터베이스 접속
   * application.properties나 yml파일에 jdbc:h2:tcp://localhost/~/db명  으로 입력해서 접속이 가능하다.



---
# Trouble Shooting

### database파일 생성오류

 * h2 재기동
 * 재기동시에 접속되는 브라우저의 URL에서 localhost로 변경후
 * jdbc:h2:~/<디비명> URL 접속하면 C:\Users\songw 경로에 디비 파일이 생성됨
 * jdbc:h2:~/<디비명> 로 접속하면, 파일에 바로 접속하게 됨.

### database생성후에는 tcp접속으로 할 것
 * jdbc:h2:tcp://localhost/~/datajpa 처럼 URL을 변경할 것.
 * 파일에 바로 접근하면 파일에 lock걸려서, 한곳에서밖에 동작이 안된다고 함.

### 메소드 이름으로 쿼리 생성시

아래 와 같은 쿼리 메소드를 작성하였다.
```
List<Order> getAllByUser_IdAndStatusNotOrderByIdDesc(Long id, OrderStatus status);
```

 * 실제 쿼리는 아래처럼 동작했다.
```SQL
    where
        user1_.user_id=4649
        and order0_.status<>'WAIT'
    order by
        order0_.order_id desc
```


### drop table 이 실패하여서 hibernate-entitymanager 버전을 올려서 해결함.
 * 아래 이슈에서 수정됨
 * https://github.com/hibernate/hibernate-orm/pull/3093

 * hibernate-entitimanager를 5.4.33.Final 버전으로 업그레이드 해서 해결함.
```xml
	<dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.4.33.Final</version>
    </dependency>
```
