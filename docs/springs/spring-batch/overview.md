---
layout: default
title: Spring Batch Overview
grand_parent: Springs
parent: Spring Batch
nav_order: 1
---

모놀리스환경에서 스케줄로 돌리는 경우(Spring은 @Scheduled, linux의 크론탭)에 배포중에 스케줄 동작을 제어하는 것이 불가능했다.

* CI/CD 배포중에는 배치가 동작하지 않도록 가능할까.
  + 배치중에 배포가 동작하지 않고, 	pipeline에서 배치처리 단계에서 java program에 통지하고, java program 내부에서 상태를 확인해서 배포중이면, pipeline에서 에러가 발생하도록 ? 하면 될 수는 있다.
  + 하지만, 다른 이유로도 배포가 실패하는 경우에도 서비스가 복구가 되지 않는 경우가 있다.(aws codepipeline의 특성; 간혹 EC2로 Linear - HalfAtATime - 로 배포하는 경우에, 배포중에 실패하고 서비스가 안살아나는 경우가 있었다)
  + 결론) 배치는 독립적인 서비스로 뺴는게 여러모로 좋다.



# 리소스
- https://docs.spring.io/spring-batch/docs/ <- 모든 매뉴얼
- https://docs.spring.io/spring-batch/docs/current <- 최신 버전

 * https://jojoldu.tistory.com/category/Spring%20Batch <= 스프링 배치관련해서 실제 테스트해보면서 정리하였고, 실무도 하고 있어서 도움이 많이 되는 듯.
 * https://docs.spring.io/spring-batch/docs/current/reference/html/transaction-appendix.html#transactions
 * https://stackoverflow.com/questions/30458437/how-does-spring-batch-manage-transactions-with-possibly-multiple-datasources
   + multi datasource에서의 트랜잭션 관리 방법에 대한 질문인데, 배치 작성하다보면 이런 궁금증이 생긴다.
 * https://docs.spring.io/spring-batch/docs/current/reference/html/

 * Transactions
     * [기본](https://blog.codecentric.de/en/2012/03/transactions-in-spring-batch-part-1-the-basics/)
     * [Restart, Cursor 기반 Reader, Listeners](https://blog.codecentric.de/en/2012/03/transactions-in-spring-batch-part-2-restart-cursor-based-reading-and-listeners/)
     * [Skip And Retry](https://blog.codecentric.de/en/2012/03/transactions-in-spring-batch-part-3-skip-and-retry/)

# Spring MVC와 같이 사용하는 경우
 * Spring Batch in Spring MVC
   * http://www.mobabel.net/spring-batch-in-spring-mvc/
   * Spring MVC에서 Spring Batch도 작성한다면!?

 * Spring Boot MVC/Batch Multi Datasource Resources
   * https://github.com/jojoldu/spring-boot-multi-datasource
   + -> 설명자료도 없고, 별로 도움이 안될듯 하다.


# 지원 Reader & Writer

### Spring Batch 4.0 (Spring Boot 2.0)

| DataSource | 기술      | 설명                                       |
|------------|-----------|--------------------------------------------|
| Database   | JDBC      | 페이징, 커서, 일괄 업데이트 등 사용 가능   |
| Database   | Hibernate | 페이징, 커서 사용 가능                     |
| Database   | JPA       | 페이징 사용 가능 (현재 버전에선 커서 없음) |
| File       | Flat file | 지정한 구분자로 파싱 지원                  |
| File       | XML       | XML 파싱 지원                                           |


---
# 메타테이블 관련
배치 실행관련 정보가 저장된다.

# JOB EXECUTION
아래처럼 설정된 경우
```java
@Slf4j // log 사용을 위한 lombok 어노테이션
@RequiredArgsConstructor // 생성자 DI를 위한 lombok 어노테이션
@Configuration
public class SimpleJobConfiguration {
...

    @Bean
    @JobScope
    public Step simpleStep1(@Value("#{jobParameters[requestDate]}") String requestDate) {
        return stepBuilderFactory.get("simpleStep1")
                .tasklet((contribution, chunkContext) -> {
                    log.info(">>>>> This is Step1");
                    log.info(">>>>> requestDate = {}", requestDate);
                    return RepeatStatus.FINISHED;
                })
                .build();
    }
}
```

IDE 실행환경에서 'program argument'를 'requestDate=20230501' 처럼 지정해서 실행할 수 있다.

같은 파라미터로 2번 실행하는 경우는 아래 에러가 나온다.
```
A job instance already exists and is complete for parameters={requestDate=20180805}.  If you want to run this job again, change the parameters.
```

**같은 Job이 Job Parameter가 달라지면 그때마다 BATCH_JOB_INSTANCE에 생성되며, 동일한 Job Parameter는 여러개 존재할 수 없습니다 .**

하지만, 만약에 같은 Job이 프로그램 오류등으로 실패한 경우에는 재실이 가능하며, 아래와 같은 실행이력을 가지게 된다.
![](/images/spring-batch/metadata-same-job-failed.png)


### Batch Status VS Exit Status
BatchStatus는 Job 또는 Step 의 실행 결과를 Spring에서 기록할 때 사용하는 Enum

BatchStatus로 사용 되는 값은 COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN


ExitStatus는 Step의 실행 후 상태
```
.on("FAILED").to(stepB())
```

위 코드에서 on 메소드가 참조하는 것은 BatchStatus 으로 생각할 수 있지만 실제 참조되는 값은 Step의 ExitStatus이다.

Spring Batch는 기본적으로 ExitStatus의 exitCode는 Step의 BatchStatus와 같도록 설정이 되어있다.

하지만 만약에 본인만의 커스텀한 exitCode가 필요하다면 어떻게 해야할까?
(즉, BatchStatus와 달라야하는 상황...)

```
.start(step1())
    .on("FAILED")
    .end()
.from(step1())
    .on("COMPLETED WITH SKIPS")
    .to(errorPrint1())
    .end()
.from(step1())
    .on("*")
    .to(step2())
    .end()
```
위 step1의 실행 결과는 다음과 같이 3가지가 될 수 있습니다.

step1이 실패하며, Job 또한 실패하게 된다. step1이 성공적으로 수행되어 step2가 수행된다.
step1이 성공적으로 완료되며, COMPLETED WITH SKIPS의 exit 코드로 종료 된다.
위 코드에 나오는 COMPLETED WITH SKIPS는 ExitStatus에는 없는 코드이다.
원하는대로 처리되기 위해서는 COMPLETED WITH SKIPS exitCode를 반환하는 별도의 로직이 필요하다.
```
public class SkipCheckingListener extends StepExecutionListenerSupport {

    public ExitStatus afterStep(StepExecution stepExecution) {
        String exitCode = stepExecution.getExitStatus().getExitCode();
        if (!exitCode.equals(ExitStatus.FAILED.getExitCode()) &&
              stepExecution.getSkipCount() > 0) {
            return new ExitStatus("COMPLETED WITH SKIPS");
        }
        else {
            return null;
        }
    }
}
```
위 코드를 설명하면 StepExecutionListener 에서는 먼저 Step이 성공적으로 수행되었는지 확인하고, StepExecution의 skip 횟수가 0보다 클 경우 COMPLETED WITH SKIPS 의 exitCode를 갖는 ExitStatus를 반환한다.

### Decide
바로 위의 방식의 문제점

* Step이 담당하는 역할이 2개 이상
  + 실제 해당 Step이 처리해야할 로직외에도 분기처리를 시키기 위해 ExitStatus 조작이 필요하다
* 다양한 분기 로직 처리의 어려움
  + ExitStatus를 커스텀하게 고치기 위해선 Listener를 생성하고 Job Flow에 등록하는 등 번거로움

Step들간의 Flow 분기만 담당하면서 다양한 분기처리가 가능한 타입이 있으면 편하겠죠?
그래서 Spring Batch에서는 Step들의 Flow속에서 분기만 담당하는 타입이 있다.

아무리 복잡한 분기로직이 필요하더라도 Step과는 명확히 역할과 책임이 분리된채 진행할 수 있게 해준다.

사용방법은 예제(job/flow/)를 확인할 것.


---
# Scope

SpEL사용예
```
@Value("#{jobParameters[파라미터명]}")
```

SpEL로 사용할 수 있는 정보
- jobParameters
- jobExecutionContext
- stepExecutionContext

Job Parameter의 타입으로 사용할 수 있는 것은 Double, Long, Date, String

LocalDate와 LocalDateTime이 없어서 String으로 받아 타입변환을 해서 사용

### 간략 소개
Spring Bean의 기본 Scope는 singleton인데, JobScope, StepScope를 이용하면 Scope의 실행시점에 해당 컴포넌트를 Spring Bean으로 생성한다.

@StepScope를 이용하면, Step의 실행시점에 해당 컴포넌트를 Spring Bean으로 생성하고

@JobScope를 이용하면, Job의 실행시점에 Bean이 생성되는데,

Bean의 생성 시점을 지정된 Scope가 실행되는 시점으로 지연시키는 것이다.

> MVC의 request scope와 약간 비슷함
request scope가 request가 왔을때 생성되고, response를 반환하면 삭제되는것처럼, JobScope, StepScope 역시 Job이 실행되고 끝날때, Step이 실행되고 끝날때 생성/삭제가 이루어진다고 보면 된다.


> Job Parameters는 Step이나, Tasklet, Reader 등 Batch 컴포넌트 Bean의 생성 시점에 호출할 수 있지만, 정확히는 Scope Bean을 생성할때만 가능하다.
즉, @StepScope, @JobScope Bean을 생성할때만 Job Parameters가 생성되기 때문에 이것을 고려해야 한다.

=> **JobParameters를 사용하기 위해선 꼭 @StepScope, @JobScope로 Bean을 생성해야한다**

### JobParameter vs 시스템 변수
왜 꼭 Job Parameter를 써야하지?

기존에 Spring Boot에서 사용하던 여러 환경변수 혹은 시스템 변수를 사용하면 되지 않나?

1. 시스템 변수를 사용할 경우 Spring Batch의 Job Parameter 관련 기능을 못쓰게 된다.
   Late Binding도 못쓰게 됨.
2. Command line이 아닌 다른 방법으로 Job을 실행하기가 어렵게 된다.



> @JobScope는 Step선언문에서 시용이 가능하고 , @StepScope는 Tasklet이나 ItemReader, ItemWriter, ItemProcessor에서 사용할 수 있다.

### JobScope
아래 정보 이용 가능
- jobParameters
- jobExecutionContext

Step 선언문에서 사용 가능

### StepScope
아래 정보 이용 가능
- stepExecutionContext

Tasklet이나 ItemReader, ItemWriter, ItemProcessor 에서 사용 가능


---
# 실행 옵션

### 설정파일

src/main/resources/application.yml
```
spring.batch.job.names: ${job.name:NONE}
```
지정한 Batch Job만 실행되도록 하는 설정이다.
이 코드의 의미는 job.name이 있으면 job.name값을 할당하고, 없으면 NONE이 할당됨.
중요한 것은! spring.batch.job.names에 NONE이 할당되면 어떤 배치도 실행하지 않는다. 즉, 혹시라도 값이 없을때 모든 배치가 실행되지 않도록 막는 역할이다.

### program argument
program argument로 지정할 수도 있다.
```
--job.name=stepNextJob
```


---
# 초기 프로젝트 세팅
start.spring.io에서 생성한다.
- Lombok
- JPA
- MySQL
- H2
- Batch
JPA만 쓰고 있다면 JDBC 선택안해도 됨.


### 스키마 생성 방법에 대해서
프로젝트 설정에서 'schema-mysql.sql'를 검색해서 DB스키마를 생성할 수도 있다.

또, 설정파일에서 아래처럼 설정하면 자동 생성해준다.
```
spring:
  batch:
    jdbc:
    initialize-schema: always
```

# 테스트 코드 작성시
H2로 작성을 하면, 메타데이터 내용을 확인할 수 없어서 어려움을 많이 겪을 수 있다. 처음에는 MySQL을 이용해서 메타데이터를 확인하면서 진행하는 것이 좋다.


# 실습 내용
도커로 MySQL을 생성하는 내용

```
// 리눅스에서(volume 설정하는 옵션)
sudo docker run -d -p 13306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql-springbatch -v /home/systemv/mysql/data:/var/lib/mysql mysql:5.7

// 윈도우즈에서(volumn 설정하지 않는 옵션)
docker run -d -p 13306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql-springbatch mysql:5.7

 - mysql container접속(linux)
sudo docker exec -i -t  mysql-springbatch bash

 - mysql container접속(windows)
winpty docker exec -i -t  mysql-springbatch bash

 - 컨테이너 안에서 저장
mysql -u root -p

 - HOST머신에서 접속할 때
 mysql -uconnectuser -p  -h 192.168.0.144   // connect123!@#
 mysql -uroot -p -h 192.168.0.144
```


```
create database springbatch;



 alter database guestbook default character set = utf8;
 create user connectuser@'%' identified by 'connect123!@#';
 create user connectuser@localhost identified by 'connect123!@#';
 grant all privileges on guestbook.* to connectuser@'%';
 grant all privileges on guestbook.* to connectuser@'localhost';
 flush privileges;


create table guestbook(
id bigint(20) unsigned not null AUTO_INCREMENT,
name varchar(255) NOT NULL,
content text,
regdate datetime,
PRIMARY KEY (id)
);

create table log(
id bigint(20) unsigned not null AUTO_INCREMENT,
ip varchar(255) NOT NULL,
method varchar(10) NOT NULL,
regdate datetime,
PRIMARY KEY (id)
);
```
