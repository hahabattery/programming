---
layout: default
title: Overview
parent: Design Patterns
nav_order: 4
---

프로그램의 내부구조에 대한 내용.

프로그램들 간의 구조에 대해서는 "Architecture" 리포지토리지에서 정리중.

iluwata/java-design-patterns 를 fork해서 확인하자(이유: 테스트케이스가 잘 작성되어있음)

bethrobson/Head-First-Design-Patterns 같은 경우는 소스 snippet으로 관리되고 있다. 이것을 fork해서 테스트케이스를 생성하려고 하였으나, iluwata/java-design-patterns에서 테스트케이스 작성을 해놓고 있어서, 진행하지 않으려고함.

# Resource
* [Software Design Principles](https://java-design-patterns.com/principles/)
* [available design patterns](https://java-design-patterns.com/patterns/)
* [Design Patterns in guru](https://refactoring.guru/design-patterns)

# diagram
객체 간의 의존성을 시각화할 수 있다.

* 연관 관계(Association)

```java
class A {
    private B b;
}
```

* 의존 관계(Dependency)

```java
class A {
    private B method(B b) {
        return new B();
    }
}
```

* 일반화 관계(Generalization)

```java
class A extends B {
}
```

* 실체화 관계(Realization)

```java
class A implements B {
}
```

# facade and spring's slf4j
SLF4J stands for Simple Logging Facade for Java. It provides a simple abstraction of all the logging frameworks. It enables a user to work with any of the logging frameworks such as Log4j, Logback, JUL (java.util.logging), etc. using single dependency.

# DI and Design Pattern
![](/images/programming-design/design-patterns-categorized.webp)
Scope > Class 는 상속을 이용한 디자인 패턴이다.
Scope > Object 은 합성을 이용한 디자인 패턴이다.
클래스를 이용한 패턴의 경우는 숫자가 적고, 합성을 이용한 디자인 패턴이 대다수이다.
오브젝트 합성을 이용하는 디자인 패턴을 적용할 때는 스프링의 의존관계 주입(Dependency Injection)을 사용해야지 디자인 패턴 이용이 손쉽다.


# 디자인패턴 적용하는 방법

### java-design-patterns.com (iluwata)
1. Search for a specific pattern by name. Can't find one? Please report a new pattern here.
2. Using tags such as Performance, Gang of Four or Data access.
3. Using pattern categories, Creational, Behavioral, and others.


# Should I be an expert at all Design Patterns?
- My personal view : Design Patterns are good to know. Have a good idea on what each one of them does. But, that where it ends. I’m not a big fan of understanding the intricate details of each Design Pattern. You can look it up if you have a good overall idea about Design Patterns.
- Full Video on Design Patterns : https://www.youtube.com/watch?v=0jjNjXcYmAU


# 디자인 패턴간의 비교

### 데코레이터 VS 어댑터 VS 퍼사드

* 데코레이터
인터페이스는 바꾸지 않고 책임(기능)만 추가한다.
* 어댑터
하나의 인터페이스를 다른 인터페이스로 변환한다.
* 퍼사드
인터페이스를 간단하게 변경한다.

### 템플릿 메소드 vs 전략 VS 팩토리 메소드
* 템플릿 메소드
알고리즘의 어떤 단계를 구현하는 방법을 서브클래스에서 결정한다.
* 전략 패턴
바꿔 쓸 수 있는 행동을 캡슐화하고, 어떤 행동을 사용할지는 서브클래스에 맡긴다.
* 팩토리 메소드
구상 클래스의 인스턴스 생성을 서브클래스에서 결정한다.

### 어댑터 vs 퍼시스
어댑터 패턴은 인터페이스를 변경해서 클라이언트에서 필요로 하는 인터페이스로 적응시키는 용도로 쓰입니다. 반면 퍼
사드 패턴은 어떤 서브시스템에 대한 간단한 인터페이스를 제공하는 용도로 쓰이죠

### 데코레이터와 프록시
종종 프록시와 데코레이터 패턴이 똑같아 보이기도 합니다. 하지만 용도로 구분할 수 있습니다. 데코레이터는 클래스에 새로운 행동을 추가하는 용도로 쓰이지만 프록시는 어떤 클래스로의 접근을 제어하는 용도로 쓰이죠.

### 어댑터와 프록시
프록시와 어댑터는 모두 클라이언트와 다른 객체 사이에서 클라이언트의 요청을 다른 객체에게 전달하는 역할을 합니다. 어댑터는 다른 객체의 인터페이스를 바꿔 주지만, 프록시는 똑같은 인터페이스를 사용한다는 차이점이 있죠.

그리고 어댑터와 정말 비슷한 보호 프록시(Protection Proxy)도 있습니다. 보호 프록시는 클라이언트의 역할에 따라서 객체에 있는 특정 메소드로의 접근을 제어합니다. 그러다 보니 보호 프록시는 클라이언트에게 인터페이스의 일부분만을 제공합니다.

이런 점은 어댑터와 비슷하다고 할 수 있죠. 보호프록시는 잠시 후에 살펴보겠습니다.
