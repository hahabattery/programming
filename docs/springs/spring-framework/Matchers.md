---
layout: default
title: Matcher
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---

Matcher 관련 내용을 정리한다. 분류를 어떻게 할지 미정이라서 별도 파일로 정리함.


* antMatchers("/test")는 정확한 /test URL만 일치.
* mvcMatchers("/test")는 /test, /test/, /test.html, /test.xyz 등 다양하게 일치.

Generally mvcMatcher is more secure than an antMatcher. and therefore is more general and can also handle some possible configuration mistakes.

mvcMatcher uses the same rules that Spring MVC uses for matching (when using @RequestMapping annotation).

> If the current request will not be processed by Spring MVC, a reasonable default using the pattern as a ant pattern will be used.

It may be added that mvcMatchers API (since 4.1.1) is newer than the antMatchers API (since 3.1).
