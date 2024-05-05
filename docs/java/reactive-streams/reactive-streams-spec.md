---
layout: default
title: Reactive Streams Specification
description: Summarizes the content related to the Reactive Stream Specification.
grand_parent: Java
parent: Reactive Streams
nav_order: 1
---

# Reactive Streams Specification
{: .no_toc}

# Table of contents
{: .no_toc .text-delta }

1. TOC 
{:toc}


> The purpose of Reactive Streams is to provide a standard for asynchronous stream processing with non-blocking backpressure.


# Resources
* https://reactive-streams.org



# 기본적인 튜토리얼
https://www.baeldung.com/java-9-reactive-streams

https://nesoy.github.io/articles/2021-09/Reactive-Stream


https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1/

https://howtodoinjava.com/spring-webflux/spring-webflux-tutorial/

https://jistol.github.io/software%20engineering/2018/04/11/observer-pubsub-pattern/


* [Observer Pattern (관찰자, 옵저버, JDK의 Observable, 발행구독과 차이점)](https://sjh836.tistory.com/180)
* [Reactive Streams (관찰자 결합, 반복자 결합, Back Pressure, 흐름, Processor, 비동기 및 병렬화 구현방식)](https://sjh836.tistory.com/182) <= 리액티브 스트림을 기본적으로 이해해야, Reactor 도 수월하게 이해할 수 있다.



# Reactive Stream이 필요한 이유
기존에는 Thread Pool을 활용해서 일처리를 진행했다.

I/O 작업이 많은 일을 하게 되는 Thread 경우 대부분 기다리는 상황이 벌어진다.

물론 다른 Thread로 제어권이 넘어가 일을 하지만 Thread Pool의 크기는 정해져 있다보니 성능에 한계가 찾아온다.

Thread Pool Size보다 더 많은 사용자의 요청이 온다면 Thread Pool Hell인 상황
위 문제를 해결하기 위해 Thread Pool 대신 비동기 & non-blocking 모델을 사용해서 메시지 커뮤니케이션으로 전환하게 된다.

이를 Reactive Stream이라고 부른다.

# Reactive Stream의 처리 방식

* I/O를 기다리지 않기 위해 적용한 Observer pattern으로 시작된다.
  + 이를 통해 Thread들을 역할별로 분리할 수 있다.
* Observer Pattern의 단점은 없을까?
  + Observer - Subject간의 서로 의존하는 상황
  + 다양한 토픽들을 사용하려면 별도로 생성해야 하는 상황

* 서로 의존하지 않기 위해 중간에 Message Broker를 추가하여 Pub-sub Pattern이 등장하게 된다.
  + Pub, Sub 서로 의존하지 않아도 되는 상황
  + 이를 통해 Topic based로 개선되어 다양한 토픽들을 다양한 구독자들에게 전달할 수 있게 된다.

![](/images/java/pub-sub.png)

* Reactive Stream 명세에 따르면 총 4가지 인터페이스로 구성되어있다.
  + Publisher
  + Subscriber
  + Subscription
  + Processor

리액티브 스트림즈는 Publisher를 이용해서 스트림을 정의하며 Subscriber를 이용해서 발생한 신호를 처리한다. Subscriber가 Publisher로부터 신호를 받는 것을 구독이라고 한다. 

* 시퀀스
리액터는 스트림이라는 용어 대신 시퀀스라는 용어를 주로 사용한다. seq라는 변수명이 자주 나온다.

# Flow Control Problem(back pressure)
* Push 방식(빠른 Publisher & 느린 Subscriber)
  + Publisher가 초당 100개의 메시지를 생산해 Subscriber에게 보낸다.
  + Subscriber은 초당 10개밖에 소비를 하지 못한다.
  + 그럼 나머지 초당 90개는 어디에 쌓아두어야 할까?
    - 고정된 버퍼를 가지고 있다면? -> 버퍼 크기를 넘어서면 버리거나 OOM 발생하거나..

* 이를 해결하기 위해 Dynamic Pull 방식으로 바꾼다.
  + Subscriber은 자신이 소비할 수 있는 크기만큼 Publisher에게 요청한다.
  + Publisher는 요청받은 메시지 크기만큼 Subscriber에게 전달한다.
  + Subscriber는 자신이 소화할 수 있는 만큼 메시지를 받기 때문에 처리하는데 문제가 없다.

* 남은 궁금증?
  + 그럼 Publisher가 넘치는 상황은 어떻게 핸들링하는가?
  + 같은 Topic에 다른 Subscriber의 소비속도가 다르면 이를 어떻게 핸들링할까?


# Reactive Stream은 Spec이고, 이를 구현한 구현체들 목록
* RxJava, Reactor Core, Akka Streams
* ReactiveMongo, Slick
* Armeria, Vert.x, Play Framework, Spring WebFlux
