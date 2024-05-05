---
layout: default
title: SpringBoot2 Releases
description: Summarizing the differences between Spring Boot releases by version.
parent: Springs
nav_order: 21
---

# v1 -> v2 마이그레이션 



# v2.5 에서 변경점

### continue-on-error 설정 방법이 달라짐
2.5부터는 아래 방법으로 설정이 가능하다.
```
spring.sql.init.continue-on-error=true
```

According to the Spring Boot 2.5.5 user guide:

> By default, Spring Boot enables the fail-fast feature of its script-based database initializer. This means that, if the scripts cause exceptions, the application fails to start. You can tune that behavior by setting spring.sql.init.continue-on-error.

* Before Spring Boot 2.5 the property was named spring.datasource.continue-on-error

