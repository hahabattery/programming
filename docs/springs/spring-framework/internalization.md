---
layout: default
title: Internalization
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---

# Spring이 지원하는 LocaleResolver

### AcceptHeaderLocaleResolver
스프링 설정 파일에 LocaleResolver가 따로 등록된게 아니라면 기본값

브라우저에서 전송된 HTTP 요청 헤더에서 Accept-Language에 설정된 Locale로 메시지를 적용함

### CookieLocaleResolver
Cookie 에 저장된 Locale 정보를 추출해서 메시지를 적용

### SessioLocaleResolver
HttpSession에 저장된 Locale 정보를 추출해서 메시지를 적용

* https://m.blog.naver.com/jjdo1994/222316772482
  + SessionLocaleResolver와 LocaleChangeInterceptor 이용예

### FixedLocaleResolver
웹 요청과 무관하게 특정 Locale로 고정함

---
# ko_KR

* Accept-Language 헤더 이용방법에 대해서 확인하기 (처음 두 글자 _ 대문자 두 글자 의 차이와 사용법).
  + 일반적으로 리소스 파일에는 'ko'와 'ko_KR'라는 문자열이 붙는다. 어떤 차이가 있을까? 이 내용은 Locale과 관련있다. Locale은 사용자 인터페이스에서 사용되는 언어, 지역, 출력 형식 등을 의미한다. ('ko'는 ISO 639-1 표준을 따르고 'KR'은 ISO 3166-1 표준 형식을 따른다고 한다.)
  + 앞에는 language, 뒤에는 territory


---
# 시스템 설정

시스템OS의(예를들면 MacOS) 언어 설정에 따라서 Spring의 다국어 설정이 동작하였다. 주언어가 English 보조 언어가 Kr인 경우는 messages_en_kr이라는 혼종으로 동작하였다.



```java
LocaleContextHolder.getLocale()
```
이럴 때는 위 명령어로 서버가 동작화는 언어 로케일 환경을 찍어보는게 도움이 될 수 있다.
