---
layout: default
title: Spring Webflux
description: 
grand_parent: Springs
parent: Spring Webflux
nav_order: 10
---

# Could it use spring-boot-starter-web and webflux together?

As explained in the [Spring Boot reference documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.web-environment) section about the web environment, adding both web and webflux starters will configure a Spring MVC web application.

But it can be acheived by following configuration.

```
spring.main.web-application-type=reactive
```


