---
layout: default
title: Trouble Shooting
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---

운영중에 발생하는 여러 이슈들

# Failed to stop bean 'webServerStartStop'

```
java.lang.NoClassDefFoundError: org/springframework/boot/web/server/GracefulShutdownResult
```

[NoClassDefFoundError 와 Spring Boot 의 Executable Jar 관계](https://yang1s.tistory.com/m/29)


>jar 파일이 변경되기 전에 이미 Load 된 class 나 jar 파일이면 상관이 없지만, 아직 Load 가 안된 상태에서 jar 파일이 변경된 경우 그리고 이로 인해 각 class 와 jar 파일의 offset 정보가 달라졌을 경우 jar 안에 class 나 jar 파일이 존재하더라도 class 를 찾을 수 없는 상태가 되기 때문에 NoClassDefFoundError, ClassNotFoundException 이 발생하게 됩니다

 * reference
   + https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix.executable-jar

# 조회 빈이 2개 이상 일 때 해결 방법
https://prodo-developer.tistory.com/121


