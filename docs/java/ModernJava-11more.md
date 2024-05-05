---
layout: default
title: Modern Java 11 more
parent: Java
nav_order: 10
---

Java 10이후 버전에 대한 요약


* 검색 키워드
  + JDK 버전별 변화

# 타입추론(JDK version 10 부터)
Java도 타입추론이 된다!

타입추론은 말그대로 개발자가 변수의 타입을 명시적으로 적어주지 않고도, 컴파일러가 알아서 이 변수의 타입을 대입된 리터럴로 추론하는 것이다.

대표적인 타입추론 언어는 자바스크립트, 코틀린, 스위프트가 있다.

> Var는 초기화값이 있는 지역변수 (Local Vairable)로만 선언이 가능하다.
> This feature is available only for local variables with the initializer.

### 타입추론이 유용한 케이스 - 람다

이 람다식에
```
Consumer<String> testFoo = s -> System.out.println("s = " + s);
```

var를 추가할 수 있다.
```
Consumer<String> testFoo = (var s) -> System.out.println("s = " + s);
```

이렇게 변경하면 키워드 앞에만 사용할 수 있는 어노테이션을 사용할 수 있다.
```
Consumer<String> testFoo = (@Nonnull var s) -> System.out.println("s = " + s);
```
