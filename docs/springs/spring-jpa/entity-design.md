---
layout: default
title: Entity Design
grand_parent: Springs
parent: Spring JPA
nav_order: 6
---


# 기본키(PK)
* 리소스
  + https://gmlwjd9405.github.io/2019/08/12/primary-key-mapping.html

* 직접 할당시는 @Id 만 사용하면 된다.
* 자동 생성시는 @Id와 @GeneratedValue를 같이 사용

### IDENTITY 전략
```
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

기본 키 생성을 데이터베이스에 위임

즉, id 값을 null로 하면 DB가 알아서 AUTO_INCREMENT 해준다.

Ex) MySQL, PostgreSQL, SQL Server DB2 등

* IDENTITY 전략은 entityManager.persist() 시점에 즉시 INSERT SQL을 실행하고 DB에서 식별자를 조회한다.
  + JPA는 보통 트랜잭션 commit 시점에 INSERT SQL을 실행한다.
  + 추가 설명...
    + IDENTITY 전략은 id 값을 설정하지 않고(null) INSERT Query를 날리면 그때 id의 값을 세팅한다.
      + AUTO_INCREMENT는 DB에 INSERT SQL을 실행한 이후에 id 값을 알 수 있다.
      + 즉, id 값은 DB에 값이 들어간 이후에서야 알 수 있다는 것이다.
    + Q. id 값을 DB에 값이 들어간 이후에 알게 됐을 때의 문제점?
      + 영속성 컨텍스트에서 해당 객체가 관리되려면 무조건 pk 값이 있어야 한다.
      + 하지만 이 경우 pk 값은 DB에 들어가봐야 알 수가 있다.
      + 다시 말해서, IDENTITY 전략의 경우 영속성 컨텍스트의 1차 캐시 안에 있는 @Id 값은 DB에 넣기 전까지는 세팅을 할 수 없다는 것이다. (JPA 입장에서는 Map의 key 값이 없으니까 해당 객체의 값을 넣을 수 있는 방법이 없다.)
    + A. 해결책?
      + IDENTITY 전략에서만 예외적으로 entityManager.persist()가 호출되는 시점에 바로 DB에 INSERT 쿼리를 날린다. (다른 전략에서는 이미 id 값을 알고 있기 때문에 commit 하는 시점에 INSERT 쿼리를 날린다.)
      + 위의 과정을 통해 entityManager.persist()가 호출되자마자 INSERT SQL을 통해 DB에서 식별자를 조회하여 영속성 컨텍스트의 1차 캐시에 값을 넣는다. (SELECT 문을 다시 날리지 않아도 된다.)
  + 단점: 모아서 INSERT 하는 것이 불가능하다.
    + 하지만, 버퍼링해서 Write 하는 것이 큰 이득이 있지 않기 때문에 크게 신경쓰지 않아도 된다.
    + 하나의 Transaction 안에서 여러 INSERT Query가 네트워크를 탄다고 해서 엄청나게 비약적인 차이가 나지 않는다.



### SEQUENCE 전략
```
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```
* 데이터베이스 Sequence Object를 사용
  + DB Sequence는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트
  + 테이블 마다 시퀀스 오브젝트를 따로 관리하고 싶으면 @SequenceGenerator에 sequenceName 속성을 추가한다.
* 즉, DB가 자동으로 숫자를 generate 해준다.
  + Ex) Oracle, PostgreSQL, DB2, H2 등
* @SequenceGenerator 필요


SEQUENCE 전략은 id 값을 설정하지 않고(null) generator에 매핑된 Sequence 전략(“MEMBER_SEQ”)에서 id 값을 얻어온다.
해당 Sequence Object는 DB가 관리하는 것이기 때문에 DB에서 id 값을 가져와야 한다.

* Q1. id 값을 DB에 값이 들어간 이후에 알게 됐을 때의 문제점?
  + 위의 IDENTITY 전략과 마찬가지의 상태
* A1. 해결책?
  + 1) entityManager.persist()를 호출하기 전에 DB의 Sequence에서 pk 값을 가져와야 한다.
    + hibernate: call next value for MEMBER_SEQ이 수행됨.
    + 현재의 값: member.id = 1
  + 2) DB에서 가져온 pk 값을 해당 객체의 id 에 넣는다.
  + 3) 이후에 entityManager.persist()를 통해 영속성 컨텍스트에 해당 객체가 저장되는 것이다.
    + 이 상태에서는 아직 DB에 INSERT 쿼리가 날라가지 않고, 영속성 컨텍스트에 쌓여있다가 트랜잭션 commit 하는 시점에 INSERT 쿼리가 날라간다.
  + 4) 필요한 경우 버퍼링이 가능하다.
    + IDENTITY 전략에서는 INSERT 쿼리를 날려야 pk를 알 수 있었기 때문에 버퍼링이 불가능하다.
* Q2. 위의 해결책이면 계속 네트워크를 왔다갔다 해야되기 때문에 성능 상의 저하가 있지 않을까?
  + SEQUENCE 전략을 사용할 경우, Sequence를 매번 DB에서 가지고오는 과정에서 네트워크를 타기 때문에 성능 상의 저하를 가져올 수 있다.
  + 차라리 INSERT Query를 한 번에 날리는 것이 낫지 않을까?
* A2. allocationSize 속성값 (기본값: 50) 이용
  + 이를 해결하기 위한 성능 최적화의 방법으로 allocationSize 옵션을 사용한다.
  + 1) 이 옵션을 사용하면 next call을 할 때 미리 DB에 50개를 한 번에 올려 놓고 (DB는 sequence가 51로 세팅된다.) 메모리 상에서 1개씩 쓰는 것이다.
  + 2) 50개를 모두 사용하면 그 때 또 next call을 날려서 다시 50개를 올려 놓는다. (DB는 sequence가 101로 세팅된다.) 메모리에서 sequence를 가져와 51부터 사용할 수 있다.
  + create sequence MEMBER_SEQ start with 1 increment by 50
  + 예시)
```
// 1(1로 맞추기 위한 dummy 호출), 51(최적화를 위한 호출)
em.persist(member1); // next call 2번 호출
em.persist(member2); // MEM
em.persist(member3); // MEM
```
* Q3. 옵션값을 50보다 큰 수로 정하면 좋지 않나?
+ A3. 이론적으로는 더 큰 수로 설정할수록 성능은 좋아진다.
  + 하지만, 중간에 애플리케이션(웹 서버)를 내리는 시점에 사용하지 않는 seq 값이 날라간다. 즉, 중간에 구멍이 생긴다.
  + 중간의 seq 공백이 큰 문제가 되는 것은 아니지만, 그래도 낭비가 되는 것이므로 적당한 50~100 사이로 설정하는 것이 좋다.


### TABLE 전략
```
@GeneratedValue(strategy = GenerationType.TABLE)
```
키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략.
@TableGenerator 필요

* 장점
  + 모든 데이터베이스에 적용 가능
* 단점
  + 최적화 되어있지 않은 테이블을 직접 사용하기 때문에 성능상의 이슈가 있음

운영 서버에서는 사용하기에 적합하지 않다. Why? DB에서 관례로 쓰는 것이 있기 때문에

* Q. Table 전략에서 allocationSize = 50이고, 서버가 여러 대인 경우에는 문제가 없을까?
* A. 문제 없다.
  + 미리 값을 올려두는 방식이기 때문에 여러 대의 서버가 붙더라도 상관이 없다.
  + 웹 서버 10대가 동시에 호출하는 경우, 순차적으로 사용하여 값이 올라가 있을 것이다.
    + A 서버: 1~50
    + B 서버: 51~100 …

### AUTO 전략
```
@GeneratedValue(strategy = GenerationType.AUTO)
```
기본 설정 값, 방언에 따라 위의 세 가지 전략을 자동으로 지정한다.


### 이상적인 식별자 구성 전략
* PK의 제약조건
  + null이 아니다.
  + 유일하다.
  + 변하면 안된다.

**미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 그 대신 대리키/대체키를 사용하자.**
* 자연키 (Natural Key)
  + 비즈니스적으로 의미가 있는 키
  + Ex) 전화번호, 주민등록번호 등
* 대리키/대체키 (Generate Value)
  + 비즈니스적으로 상관없는 키
  + Ex) Generate Value, 랜덤 값, 유휴 값 등

**(Long형) + (대체키) + (적절한 키 생성 전략) 사용**

1. Long Type (아래 참고)
2. 대체키 사용: 랜덤 값, 유휴 ID 등 비즈니스와 관계없는 값 사용
3. AUTO_INCREMENT 또는 Sequnce Object 사용

* [참고] Q. id 값(pk 값)은 어떤 타입을 써야 할까?
  + int 0이 있다.
  + Integer 10억 정도 까지만 가능하다.
  + Long 채택! Long의 크기가 Integer의 2배지만 애플리케이션 전체로 봤을 때의 영향은 작다고 볼 수 있다. 오히려 10억이 넘어갔을 때 해당 id 값을 타입을 변경하는 것이 더 어렵다.



---
# 상속관계 엔티티 설계

엔티티 설계시에 중복이 없는 형태가 바람직한 것은 두말할 필요가 없다.

그래서 엔티티를 상속관례를 이용해서 설계하려고 하는데,

Relational databases don’t have a straightforward way to map class hierarchies onto database tables.

To address this, the JPA specification provides several strategies:



### MappedSuperclass
- **the parent classes, can’t be entities**

가정 먼저 생각할 수 있는 방법이다.

But
* 'Many To One' attribute type should not be 'Mapped Superclass'
  + JPA에서 대상 클래스는 상속 계층 구조의 일부여야 한다. Mapped Superclass는 엔티티가 아니기 때문에, 관계의 대상이 될 수 없다는 제약 때문에, 하위 엔티티 클래스를 대상으로 연관(ManyToOne OneToMany 등등)을 맺도록 작업해야 한다. 이 제약으로 인해, 상위 base 클래스에서 공통된 데이터임에도 하위 클래스에 나뉘어서 배치해서 관리해야되는 거추장스러움이 생기게 된다.

따라서
연관관계 설정을 하는 경우는 MapperSuperclass의 하위 클래스에서 따로 잡아야 하는 문제가 생기게 된다. 중복을 줄이는 목적은 달성하지만, MapperSuperclass를 이용해서 추상적으로 직관적으로 이해하기 쉬운 관계를 design하려고 하는 목적을 달성하지 못하게 된다는 생각이 든다. Inheritance전략(SINGLE_TABLE, TABLE_PER_CLASS, JOINED)의 경우에는 연관의 대상이 될 수 있기 때문에, Inheritance전략을 이용하는게 나을 것이다.

* 결론
  + MappedSuperclass는 상속관계라기 보다는 속성만 사용할 수 있도록 하는 목적(감사등의...)일 때 사용하는게 낫다.


### SINGLE_TABLE
- **The entities from different classes with a common ancestor are placed in a single table.**
- This strategy provides good support polymorphic relationships between entities and queries that cover the entire class hierarchy.
- May contain **null fields** for some subclass data

null fields가 생길 수밖에 없는 점이 이 전략의 특징인데, null이 몇개 안되는 경우에만 이 전략이 허용될 것이다.

* 전체 조회시에 SINGLE_TABLE 인 경우에 base 클래스에 대해서 조회하면 된다.


### TABLE_PER_CLASS
- **All the properties of a class are in its table, so no join is required.**
- Each class in a hierarchy mapped to a separate table and hence, provides poor support for polymorphic relationships
- 각각이 테이블로 분리되어있기 때문에 **requires SQL union** or separate SQL queries for each subclass

* **전체 조회시에 TABLE_PER_CLASS 인 경우에 base 클래스에 대해서 조회하면 union 쿼리가 실행된다**
* 중요! 데이터베이스 종류에 따라서 @Id의 GenerationType을 TABLE 타입을 사용해야 함. 또한, sequence 테이블을 이용하게 되므로, 엔티티 클래스에 sequence 테이블 매핑까지 처리해야 한다.


### JOINED
- **Each class has its table, and querying a subclass entity requires joining the tables.**
- no null fields => compact data
- This provides good support for polymorphic relationships, but requires one or more join operations – may result in poor performance

이 케이스는 테이블간의 연관 정보(FK)를 RDB에서 관리하게 되기 때문에, 초기 엔티티 설계시보다 RDB 테이블구조가 더 복잡해는 결과로 이어질 수 있다.

* 데이터 저장시에 INSERT 쿼리가 상위, 하위 테이블 두번 발생한다.
* 테이블 정규화로 저장공간 효율화가 좋다(하지만, 크게는 안될듯)
* 전체 조회시, 3가지 전략중에서 제일 복잡(상속관계 테이블간에 Join + Union)

---
# 엔티티 설계시의 딜레마

* 중복제거는 엔티티관점에서 코드의 중복뿐만 아니라, RDB 스키마 관점에서 컬럼의 중복(정규화)도 있기 때문에 균형을 잘 잡아서 설계해야한다.

* 상속을 이용해서 작업할 것인지
  + 만약에 상속을 이용시에 같은 데이터를 가지는 엔티티가 만들어진다고 하면, 상속을 이용하는게 맞나?
  + 레코드를 구분하는 type값을 가지고 구별해서 처리하는게 낫지 않나?
  + 어디까지 상속으로 처리할 것인지는 애매모호한 거 같다.


---
# 요약

### 상속관계 매핑 정리
* 기본적으로는 조인 전략을 가져가자.
* 그리고 조인 전략과 단일 테이블 전략의 trade off를 생각해서 전략을 선택하자.
* 굉장히 심플하고 확장의 가능성도 적으면 단일 테이블 전략을 가져가자. 그러나 비즈니스 적으로 중요하고, 복잡하고, 확장될 확률이 높으면 조인 전략을 가져가자.
