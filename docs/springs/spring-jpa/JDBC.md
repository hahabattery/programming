---
layout: default
title: JPA With JDBC
grand_parent: Springs
parent: Spring JPA
nav_order: 5
---

JPA와 JDBC를 함께 사용하게 되는 경우에 JDBC처리에 대해서 정리.

---
# Transaction

JpaTransactionManager 의 API 명세에서 확인할 수 있습니다.

>This transaction manager also supports direct DataSource access within a transaction (i.e. plain JDBC code working with the same DataSource). This allows for mixing services which access JPA and services which use plain JDBC (without being aware of JPA)! Application code needs to stick to the same simple Connection lookup pattern as with DataSourceTransactionManager (i.e. DataSourceUtils.getConnection(javax.sql.DataSource) or going through a TransactionAwareDataSourceProxy). Note that this requires a vendor-specific JpaDialect to be configured.
8:39

JPA 트랜잭션 매니저는 트랜잭션 내에서 동일한 DataSource로 작동하는 일반 JDBC 코드와 같은 직접적인 DataSource 액세스를 지원합니다. 이를 통해 JPA에 액세스하는 서비스와 JPA를 알지 못하는 일반 JDBC를 사용하는 서비스를 혼합 할 수 있음.

> Note: To be able to register a DataSource's Connection for plain JDBC code, this instance needs to be aware of the DataSource (setDataSource(javax.sql.DataSource)). The given DataSource should obviously match the one used by the given EntityManagerFactory. This transaction manager will autodetect the DataSource used as the connection factory of the EntityManagerFactory, so you usually don't need to explicitly specify the "dataSource" property.

참고 : 일반 JDBC 코드에 대해 DataSource의 연결을 등록하려면 Jpa 트랜잭션 매니저 인스턴스가 DataSource (setDataSource (javax.sql.DataSource))를 인식해야합니다. 주어진 DataSource는 JPA 트랜잭션 매니저의 EntityManagerFactory에서 사용하는 것과 분명히 일치해야합니다. JPA 트랜잭션 매니저는 EntityManagerFactory의 연결 팩토리로 사용되는 DataSource를 자동 감지하므로 일반적으로 "dataSource"속성을 명시적으로 지정할 필요가 없습니다.
