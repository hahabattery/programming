---
layout: default
title: Logging
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---

Logging 프레임워크는 log4j -> logback -> log4j2 순서로 등장함.

logback, log4j2는 둘 다 log4j를 기반으로 하고 있어 설정하는 방법이나 사용 방법이 유사함.

log4j는 2015년에 개발이 중단됨.

---
# logback
logback은 log4j와 유사하면서도 향상된 성능과 필터링 옵션을 제공하며, **slf4j도 지원**합니다. 자동 리로드 기능도 지원함.

### 자동 리로드

Linux 서버 내에서 log4j를 사용할 시 log level을 변경하게 되면, 서버를 재가동하여 변경 사항을 적용해줘야 하는 반면에 logback은 서버를 재가동할 필요 없이 즉각 자동 리로드를 지원해줍니다.

spring boot 환경의 경우 spring-boot-starter-web > spring-boot-starter-logging에 기본적으로 logback 구현체가 포함되어 있습니다. 그렇기 때문에 spring boot 환경에서 로깅 프레임워크를 따로 설정하지 않으면 logback이 기본으로 적용됩니다.

---
# log4j2
log4j2는 logback 이후에 나온 만큼 logback과 마찬가지로 필터링 기능과 자동 리로드 기능을 가지고 있습니다.

logback과의 가장 큰 차이점은 Multi Thread 환경에서 비동기 로거(Async Logger)의 경우 log4j, logback 보다 처리량이 더 높고, 대기 시간이 훨씬 짧습니다.

추가적으로 람다 표현식과 사용자 정의 로그 레벨도 지원합니다.


* spring boot에서 log4j2를 사용하기 위해서는 dependency에서 logback을 제거해주는 작업이 필요합니다.

스프링 부트는 기본 설정으로 logback을 사용하기 때문에 log4j2 의존성을 추가하더라도 기본 설정으로 잡힌 logback이 동작하게 됩니다.

---
# slf4j (Simple Logging Facade For Java)
slf4j는 로깅에 대한 추상 레이어로 위에서 본 logback, log4j2 같은 로깅 프레임워크의 인터페이스 역할을 하며, slf4j를 이용하면 코드를 일정하게 유지하면서 구현체의 전환 (다른 프레임워크로의 전환)을 쉽고 간단하게 할 수 있습니다.

* logback, log4j2는 slf4j의 구현체
