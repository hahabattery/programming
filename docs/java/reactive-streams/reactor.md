---
layout: default
title: Reactor Overview
description: Provides an overview of Project Reactor, an implementation of the Reactive Stream Specification.
grand_parent: Java
parent: Reactive Streams
nav_order: 2
---

# Reactor Overview
{: .no_toc}

# Table of contents
{: .no_toc .text-delta }

1. TOC 
{:toc}



Reactor has supported the Reactive Streams standard from version 2.5 milestone (February 2016). Release was marked as version 3.0 in August 2016.

Around the same time, RxJava 2.0, supporting the Reactive Streams standard, was also released. The significant difference lies in the fact that RxJava supports JDK 6 onwards, including Android, while Reactor requires JDK 8 or later.

# Resources
* [Build Reactive MicroServices using Spring WebFlux/SpringBoot
](https://github.com/dilipsundarraj1/reactive-spring-webflux/tree/final)
* [Reactor 3 Reference Guide](https://projectreactor.io/docs/core/release/reference)
* [Reactor Core library](https://projectreactor.io/docs/core/release/api/)
* [Hot Versus Cold](https://projectreactor.io/docs/core/snapshot/reference/#reactor.hotCold)
* [Threading and Schedulers](https://projectreactor.io/docs/core/release/reference/#schedulers)
* [Activating Debug Mode - aka tracebacks](https://projectreactor.io/docs/core/snapshot/reference/#debug-activate)



# 튜토리얼
* [Intro To Reactor Core](https://www.baeldung.com/reactor-core)
 * https://github.com/eugenp/tutorials/tree/master/reactor-core
   + => 코드에서 다양한 리액터 관련한 글을 볼 수 있다.

 * https://www.baeldung.com/java-reactive-systems
   + => 리액티브 하게 MSA를 구성하는 내용. 트랜잭션 관리에 대한 내용도 있음!!
   + => 리액티브 시스템을 구현하는 것에 대해서 고민해야 되는 것들(디스커버리, 오토스케일링 등)에 대해서도 언급함...

 * https://tech.kakao.com/2018/05/29/reactor-programming/
   * 간단한 예제로 flux병령처리 내용도 포함하고 있다.

 * https://javacan.tistory.com/entry/spring-reactor-intro-list
   * 가장 최근에 보던 글
  

# Publisher interface 의 구현체
[Flux](https://projectreactor.io/docs/core/release/reference/#flux) and [Mono](https://projectreactor.io/docs/core/release/reference/#mono) implemented Publisher interface. 이 둘은 대략적인 카디널리티 정보를 담는 식으로 타입을 구분한다.
If 아이템이 0개 아니면 1개라면 Mono can be used.
Mono의 연산자들은 버퍼 중복, 값비싼 동기화 작업 등이 생략해서 성능적인 이점이 있다.
But, Flux와 Mono는 서로 쉽게 변환할 수 있다, also.

```
Flux<T>.collectList() = Mono<List<T>>
Mono<T>.flux() = Flux<T>
```

publisher 는 구독자가 없으면 가만히 있기 때문에 메모리 부족을 야기하지 않고도 무한대의 리액티브 스트림을 만들 수 있습니다. publisher 인터페이스는 subscribe 라는 추상 메서드를 가집니다. Publisher.subscribe(Subscriber)의 형식으로 data 제공자와 구독자가 연결된다.

Tip 
{: .label .label-yellow } <BR>
Flux와 Mono를 직접 생성하기보다는 다른 라이브러리가 제공하는 Flux와 Mono를 사용할 때가 많다. 예를 들어 스프링 5 버전에 추가된 WebClient 클래스를 사용할 때에는 WebClient가 생성하는 Mono를 이용해서 데이터를 처리한다.

Tip 
{: .label .label-yellow } <BR>
시퀀스를 직접 생성할 일이 많지는 않다. 보통은 라이브러리(ex> grpc, kafka) 가 제공하는 기능을 사용하기 때문이다. 

### just VS fromSupplier VS defer
Flux와 Mono는 팩토리 메소드를 제공하기 때문에 햇갈려서 정리한다.

* just는 즉시 시퀀스를 만든다.
* fromSupplier과 defer는 구독시점에 시퀀스를 생성한다. (lazy 처리)
* fromSupplier 는 Supplier<? extends T> supplier 를 인자로 받는다. Mono가 아닌 값에 사용하면 된다.
* defer 는 Supplier<? extends Mono<? extends T>> supplier 를 인자로 받는다. 이미 Mono로 반환되는 메소드에 사용하는 게 좋다.

즉, 아래 경우에는 fromSupplier 가 더 적절하고 깔끔해보이는 상황이다.

```java
@Test
public void defer_테스트() throws Exception {
    long start = System.currentTimeMillis();

    // just
    Mono<Long> clock1 = Mono.just(System.currentTimeMillis());

    Thread.sleep(5000);
    long result1 = clock1.block() - start;
    log.info("흐른 시간 = {}", result1); // 5초가 지났으나, 위에서 즉발실행되어서 0 출력


    // fromSupplier
    Mono<Long> clock2 = Mono.fromSupplier(System::currentTimeMillis);

    Thread.sleep(5000);
    long result2 = clock2.block() - start;
    log.info("흐른 시간 = {}", result2); // 10초 지남


    // defer
    Mono<Long> clock3 = Mono.defer(() -> Mono.just(System.currentTimeMillis()));

    Thread.sleep(5000);
    long result3 = clock3.block() - start;
    log.info("흐른 시간 = {}", result3); // 15초 지남
}
```

```
// 실행결과
23:41:03.091 [main] INFO com.devljh.reactive.practice.ReactorTest - 흐른 시간 = 0
23:41:08.106 [main] INFO com.devljh.reactive.practice.ReactorTest - 흐른 시간 = 10108
23:41:13.128 [main] INFO com.devljh.reactive.practice.ReactorTest - 흐른 시간 = 15123
```

# Subscriber interface의 구현체

Subscriber가 구독을 신청하게 되면 Publisher로부터 이벤트를 수신받는다.

이 이벤트들은 Subscriber 인터페이스의 메서드를 통해 전송된다.

* onSubscribe
  + 구독시 최초에 한번만 호출
* onNext
  + 구독자가 요구하는 데이터의 수 만큼 호출 (최대 java.lang.Long.MAX_VALUE)
* onError
  + 에러 발생 또는 더이상 처리할 수 없는 경우
* onComplete
  + Publisher가 데이터 통지를 완료했음을 알릴 때

Subscriber가 수신할 첫 번째 이벤트는 onSubscribe 메서드의 호출을 통해 이루어진다.
Publisher가 onSubscribe 메서드를 호출할 때 이 메서드의 인자로 Subscription 객체를 Subscriber에 전달한다.

* Subscriber의 구현 예

```java
public class CustomSubscriber extends BaseSubscriber<String>
{
    @Override
    public void hookOnSubscribe(Subscription subscription) {
        System.out.println("onSubscribe, subscription " + subscription.hashCode());
        request(1); // 구독 후, 최초 요청
    }

    @Override
    public void hookOnNext(String s) {
        System.out.println("onNext = {}" + s);
        request(1); // 데이터 수신 후, 추가 요청
    }

    @Override
    public void hookOnComplete() {
        System.out.println("onComplete");
    }
}
```

리액터 프레임워크에서 미리 만들어둔, BaseSubscriber 추상클래스를 상속해서 구현한다.

Subscriber의 onSubscribe, onNext, onError, onComplete 는 모두 직접 재정의할 수 없게 final 로 선언되어 있다. 대신 제공하는 hook 을 통해서 상황에 따라 조정할 수 있도록 할 수 있다.

일반적인 Reactive Streams 에서의 Publisher 는 subscribe() 를 호출할 때 subscriber 를 등록해준다.

```java
    @Test
    void subscriber()
    {
        Flux<String> flux = Flux.just("A", "B");

        flux.subscribe(new CustomSubscriber()); // 커스텀으로 구현한 subscriber 사용
    }
```
하지만 Flux 에서 제공하는 팩토리 메서드는 위와 같이 subscriber 를 등록해주는 메서드도 있는 반면에 subscriber 를 매개변수로 등록하지 않는 함수도 존재한다.

subscriber 가 필요없어서가 아니라, 내부 로직에서 자동으로 subscriber 를 만들어 주기 때문이다.(new LambdaSubscriber<>(...) 를 통해 Subscriber를 생성)

이때 publisher에서 전달받을 개수를 지정하는 s.request는 디폴트로 Long.MAX_VALUE가 세팅된다.

```java
    @Test
    void nonSubscriber()
    {
        List<String> list = new ArrayList<>();

        Flux<String> flux = Flux.just("A", "B");
        flux.subscribe(list::add); // 내부적으로 subscriber를 구현해줌

        Assertions.assertEquals(2, list.size());
    }
```


# Subscription interface의 구현체
Subscriber는 Subscription 객체를 통해서 구독을 관리한다.
Subscription 인터페이스는 Subscriber가 구독한 데이터의 개수를 요청하거나 데이터 요청의 취소, 즉 구독을 해지하는 역할을 한다.

Subscription 인터페이스는 request 메서드와 cancel 메서드를 가지고 있다.

Publisher에서 Subscriber의 onSubscribe 메서드를 호출하며 매개변수로 subscription을 전달한다.

Subscriber는 Subscription의 request()를 호출하여 데이터를 요청하거나, cancel()을 호출하여 데이터를 더 이상 수신하지 않거나 구독을 취소할 수 있다.

request()를 호출할 때 Subscriber는 받고자 하는 데이터 항목 수를 나타내는 long 타입 값을 인자로 전달하는데 이것이 백프레셔이며, Subscriber가 처리할 수 있는 것보다 더 많은 데이터를 전송하는 것을 막아준다.

Subscriber의 데이터 요청이 완료되면 데이터가 스트림을 통해 전달되기 시작합니다. 이 때 onNext() 메서드가 호출되어 Publisher가 전송하는 데이터가 Subscriber에게 전달되며, 에러 발생시 onError()가 호출된다.

그리고 Publisher에서 전송할 데이터가 없고 더 이상 데이터를 생성하지 않는다면 Publisher가 onComplete()를 호출하여 작업이 끝났다고 Subscriber에게 알려준다.

전반적인 코드는 위의 CustomSubscriber 클래스를 확인한다.

### 푸시 모델 vs 풀 모델
Subscription#request()는 Subscriber가 데이터를 처리할 수 있을 때 Publisher에게 데이터를 요청하는 풀(pull) 모델이다. 하지만 request(Long.MAX_VALUE)로 요청하면 Publisher는 개수 제한 없이 Subscriber에 데이터를 전송한다. 이는 완전한 푸시(push) 모델이다. 또 request(100000)을 사용하면 십 만 개의 데이터를 요청하고, Publisher는 발생한 데이터가 십 만 개가 될 때까지 신호를 보낸다. 데이터 요청은 풀 모델로 이루어졌지만 10만 개의 데이터를 전송하는 동안은 실질적으로 푸시 모델과 같다.


# Cold Sequence / Hot Sequence
Cold와 Hot의 차이는 간단하게 Cold는 새로 시작한다, Hot은 새로 시작하지 않는다로 이해할 수 있다.

우리가 월간 잡지를 보내주는 서비스를 5월에 구독한다고 했을 때, 1월부터 5월까지 모든 잡지를 배달받고 이후로도 월마다 잡지를 배달받는 구조를 Cold, 5월 이후의 잡지만 배달받는 형태를 Hot이라고 보면 된다.

Cold Sequnece 흐름으로 동작하는 Publisher를 Cold Publisher라고 하고,
Publisher가 데이터를 emit하는 과정이 한 번만 일어나도 Subscriber가 각각의 구독시점 이후에 emit된 데이터만 전달받는 데이터의 흐름을 Hot Sequence라고 한다.


# Processor interface의 구현체



# 전체적인 흐름
![](/images/java/all-reactive-streams-components.png)

1. Subscriber 가 subscribe 메소드를 통해 Publisher 에게 구독을 요청
2. Publisher 는 onSubscribe 메소드로 Subscriber 에게 Subscription 를 전달
   1. Subscripotion은 구독 라이프 사이클을 관리한다.
3. Subscriber 는 Subscription.request 을 통해, 자신에게 데이터를 흘려줄 것을 요구
4. Publisher 는 Subscription 를 통해 Subscriber.onNext로 데이터를 전달한다.
   1. Subscriber 는 내부에 Subscription를 set하였기 때문 (2번)
5. 전달이 잘 끝났으면, onComplete, 오류났다면 onError 로 끝낸다.


# Processor (Publisher와 Subscriber를 혼합)
![](/images/java/reactive-sptream-components-processor.png)

Publisher 와 Subscriber 를 혼합한 Processor 라는 것도 있다.
```java
  public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
  }
```

이 둘 사이에서 몇 가지 처리 단계를 유연하게 추가할 수 있다.

하나의 subscriber 의 결과물을 다른 subscriber 에 그대로 전달하거나, 변형할 때도 사용할 수있다. 마치 새로운 Publisher 처럼 행동하는 것이다.

* ex1) 구독자 중 기준에 일치하는 구독자들에게만 전송한다던지..
* ex2) 멀티캐스팅


# Flow (JDK 9)
리액티브 스트림은 jdk9 에서도 Flow API 로 반영이 되었다.

리액티브 스트림 타입들을 Flow 로도 쉽게 변환이 가능하다. (ex. FlowAdapters)


# BackPressure on Reactor library

### request()로 요청 데이터 개수 제어
첫 번째 방법은 request()로 요청 데이터 개수 제어할 수 있다.
Subscriber가 적절히 처리할 수 있는 수준의 데이터 개수를 Publisher에게 요청할 수 있다.


Reactor에서 제공하는 Backpressure 전략이 있다.

### IGNORE 전략
IGNORE 전략은 말 그대로 백프레셔를 사용하지 않는 전략이다.

### ERROR 전략

```java
 AssertSubscriber<Integer> ts = AssertSubscriber.create();
  Flux.range(1, 10)
    .onBackpressureError()
    .subscribe(ts);
  ts.assertValues(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
   .assertNoError()
   .assertComplete();
```

에러 전략은 Downstream의 데이터 처리 속도가 느려서 Upstream의 emit 속도를 따라가지 못할 경우 IllegalStateExcetpion을 발생시킨다. 이 경우 Publisher는 에러 시그널을 Subscriber에게 전송하고 삭제한 데이터는 폐기한다.


### DROP 전략 (outside buffer)

```java
 AssertSubscriber<Integer> ts = AssertSubscriber.create();
  Flux.range(1, 10)
    .onBackpressureDrop(dropped -> log.info("dropped {}", dropped))
    .subscribe(ts);
  ts.assertValues(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
   .assertNoError()
   .assertComplete();
```

DROP전략은 Publisher가 Downstream으로 전달할 데이터가 버퍼에 가득찬 경우, 버퍼 밖에서 대기 중인 데이터 중에서 먼저 emit됐던 데이터부터 Drop시키는 전략이다. Drop된 데이터는 폐기된다.

### LATEST 전략 (outside buffer)

Pulisher가 Downstream으로 전달할 데이터가 버퍼에 가득찰 경우, 버퍼 밖에서 대기 중인 데이터는 데이터가 들어올 때마다 이전에 유지하고 있던 데이터는 폐기하고, 버퍼가 빌 때 최근에(나중에) emit된 데이터부터 버퍼에 채우는 전략이다.

DROP과 차이점이 있다면 버퍼가 가득 찰 경우 DROP은 버퍼 밖에서 대기 중인 데이터를 하나씩 차례대로 폐기한다면, LATEST는 새로운 데이터가 들어오는 시점에 가장 최근의 데이터만 남겨두고 나머지 데이터를 폐기한다.

### BUFFER 전략
버퍼가 가득차면 버퍼 내의 데이터를 폐기하거나 버퍼가 가득차면 에러를 발생시키는 전략도 지원한다.

DROP과 LATEST가 버퍼 밖의 데이터를 폐기한다면 BUFFER전략에서의 데이터 폐기는 버퍼가 가득찼을 때 버퍼 안의 데이터를 폐기하는 동작을 하는 것이 다르다. DROP_LATEST와 DROP_OLDEST 두가지가 있습니다.

##### DROP_LATEST 전략
Publisher가 Downstream으로 전달할 데이터가 버퍼에 가득 찰 경우 이후에 emit된 데이터는 Drop하는 전략

1. buffer-size=5라고 하고 버퍼에 다음과 같이 쌓여있는 경우 [5,4,3,2,1]
2. 6을 emit [6,5,4,3,2,1]
3. 이는 버퍼 사이즈를 초과해서 6을 드랍 [5,4,3,2,1]
4. 7을 emit [7,5,4,3,2,1]
5. 이는 버퍼 사이즈를 초과해서 7을 드랍 [5,4,3,2,1]

##### DROP_OLDEST 전략
DROP_LATEST와 반대로 Publisher가 Downstream으로 전달할 데이터가 버퍼에 가득 찬 상태에서 데이터가 emit될 경우, 버퍼 안에 가장 오래된 데이터부터 Drop하는 전략

1. buffer-size=5라고 하고 버퍼에 다음과 같이 쌓여있는 경우 [5,4,3,2,1]
2. 6을 emit [6,5,4,3,2,1]
3. 이는 버퍼 사이즈를 초과해서 1을 드랍 [6,5,4,3,2]
4. 7을 emit [7,6,5,4,3,2]
5. 이는 버퍼 사이즈를 초과해서 2을 드랍 [7,6,5,4,3]


# Sinks
* Processor의 기능을 개선한 Sinks가 Reactor 3.4.0 버전부터 지원
* Processor와 관련된 API는 Reactor 3.5.0부터 완전히 제거될 예정

Sinks 등장 이전에 Reactor에서 프로그래밍 방식으로 signal을 전송하는 가장 일반적인 방법은 generate() Operator나 create() Operator 등을 사용하는 것인데, 이는 싱글스레드 기반에서 signal을 전송한다. 반면, Sinks는 멀티스레드 방식으로 signal을 전송해도 스레드 안전성을 보장하기 때문에 예기치 않은 동작으로 이어지는 것을 방지해준다.

Sinks는 Sinks.Many 또는 Sinks.One interface를 사용해서 Thread-Safe하게 signal을 발생시킨다.

### Sinks.One

Sinks.One은 한 건의 데이터를 프로그래밍 방식으로 emit한다.

emit 된 데이터 중에서 단 하나의 데이터만 Subscriber에게 전달한다. 나머지 데이터는 Drop 된다.

```java
@Slf4j
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Sinks.One<String> sinkOne = Sinks.one();
        Mono<String> mono = sinkOne.asMono();

        sinkOne.emitValue("Hello Reactor", FAIL_FAST);
//        sinkOne.emitValue("Hi Reactor", FAIL_FAST);
//        sinkOne.emitValue(null, FAIL_FAST);

        mono.subscribe(data -> log.info("# Subscriber1 {}", data));
        mono.subscribe(data -> log.info("# Subscriber2 {}", data));
    }
}
```
* emitValue(): 메서드의 두 번째 파라미터는 emit 도중 에러가 발생할 경우 어떻게 처리할 것인지에 대한 핸들러를 지정한다.
* FAIL_FAST: 빠르게 실패 처리한다는 의미는 에러가 발생했을 때 재시도를 하지 않고 즉시 실패 처리를 한다는 의미이다. 주석 처리된 코드를 주석 해제하고 다시 실행하면 출력 결과는 동일하나 Drop 되었다는 디버그 로그를 확인할 수 있다. 즉, Sinks.One으로 아무리 많은 수의 데이터를 emit한다 하더라도 처음 emit한 데이터는 정상적으로 emit되지만 나머지 데이터들은 Drop된다.
* asMono(): emit한 데이터를 구독하여 전달받기 위해 Mono 객체로 변환합니다.

### Sinks.Many
Sinks.Many는 여러 건의 데이터를 프로그래밍 방식으로 emit한다.

Sinks.One은 한 건의 데이터를 emit하는 한 가지 기능만 가지기 때문에 Default Spec(SinksSpecs.DEFAULT_ROOT_SPEC)을 사용한다. 반면, Sinks.Many의 경우 데이터 emit을 위한 여러 가지 기능이 정의된 ManySpec을 리턴한다.

```java
public final class Sinks {
  ...
  ...
    public interface ManySpec {
      UnicastSpec unicast();
      MulticastSpec multicast();
      MulticastReplaySpec replay();
    }
}
```


##### UnicastSpec
UnicastSpec은 내부적으로 UnicastProcessor을 사용하며 단 하나의 Subscriber에게만 데이터를 emit을 허용한다. Subscriber가 여러 개일 경우 예외를 발생시킨다.

##### MulticastSpec
MulticastSpec의 기능은 하나 이상의 Subscriber에게 데이터를 emit 한다.

Sinks가 Publisher의 역할을 할 경우 기본적으로 Hot Publisher로 동작한다.

##### MulticastReplaySpec
MulticastReplaySpec에는 emit된 데이터를 다시 Replay해서 구독 전에 이미 emit된 데이터라도 Subscriber가 전달받을 수 있게 하는 다양한 메서드들이 정의되어 있다.

* all()
  + 구독 전에 이미 emit된 데이터가 있더라도 처음 emit된 데이터부터 모든 데이터들이 Subscriber에게 전달된다.

* limit()
  + emit된 데이터 중 파라미터로 입력한 개수만큼 뒤에서부터 되돌려 Subscriber에게 전달된다.

```java
@Slf4j
public class Main {
    public static void main(String[] args) {
        Sinks.Many<Integer> replaySink = Sinks.many().replay().limit(2);
        Flux<Integer> fluxView = replaySink.asFlux();

        replaySink.emitNext(1, FAIL_FAST);
        replaySink.emitNext(2, FAIL_FAST);
        replaySink.emitNext(3, FAIL_FAST);

        fluxView.subscribe(data -> log.info("# Subscriber1: {}", data)); // 2, 3, 4

        replaySink.emitNext(4, FAIL_FAST);

        fluxView.subscribe(data -> log.info("# Subscriber2: {}", data)); // 3, 4
    }
}
```

Subs1은 1,2,3 중 뒤에서 2칸 되돌려 2부터 재생해 2, 3, 4를 받는다.
Subs2는 1,2,3,4 중에서 뒤에서 2칸 되돌려 3부터 재생해 3, 4를 받는다.


# Scheduler
Reactor의 스케쥴러는 OS 스케쥴러와 의미가 비슷하다.
리액터에서 스케쥴러는 비동기 프로그래밍을 위해 사용되는 스레드를 관리해주는 역할을 한다.
우리는 스케쥴러를 이용하여 어떤 스레드에서 무엇을 처리할지 제어한다. 멀티스레드 환경에서 멀티스레드를 완벽하게 제어하는 일은 어렵지만 리액터 스케쥴러는 스레드 제어를 대신 해주기에 개발자가 직접 스레드를 제어해야하는 부담에서 벗어날 수 있다.

```java
    @Test
    void singleThreadTest()
    {
        Flux.range(1, 3)
            .map(i -> {
                System.out.printf("%s, map %d to %d\n", Thread.currentThread(), i, i + 2);
                return i + 2;
            })
            .flatMap(i -> {
                System.out.printf("%s, flatMap %d to Flux.range(%d, %d)", Thread.currentThread(), i, 1, i);
                return Flux.range(1, i);
            })
            .subscribe(i -> System.out.println(Thread.currentThread() + " next " + i));
    }
```

다음 테스트코드를 실행시켜보면 모두 main 스레드에서 실행되는 것을 확인하여 리액터는 비동기 실행을 강제하지 않는다는 것을 알 수 있다. 실행 결과를 보면 map(), flatMap(), subscribe()에 전달한 코드가 모두 main 쓰레드에서 실행된 것을 알 수 있다. 즉, map 연산, flatMap 연산뿐만 아니라 subscribe를 이용한 구독까지 모두 main 쓰레드가 실행하게된다.

이때 스케줄링을 위한 전용 Operator를 사용하면 구독이나 신호 처리를 별도 쓰레드로 실행할 수 있다. 대표적으로 publishOn(), SubscribeOn(), parallel()이 있으며 Operator의 파라미터로 적절한 스케쥴러를 전달하면 해당 스케쥴러의 특성에 맞는 스레드가 리액터 시퀀스에 할당된다.



### publishOn Operator 이용한 신호 처리 쓰레드 스케줄링
publishOn는 다운스트림으로 시그널을 전송할 때 실행되는 스레드를 제어하는 역할을 하는 오퍼레이터이다.

```java
    @Test
    void publishOnTest() throws InterruptedException
    {
        CountDownLatch latch = new CountDownLatch(1);

        Flux.range(1, 6)
            .map(i -> {
                System.out.printf("%s, map 1: %d to %d\n", Thread.currentThread(), i, i + 10);
                return i + 10;
            })
            .publishOn(Schedulers.boundedElastic(), 2) // 두 번째 인자인 2는 스케줄러가 신호를 처리하기 전에 미리 가져올 (prefetch) 데이터 개수이다.
            .map(i -> {     // publishOn에서 지정한 PUB 스케줄러가 실행
                System.out.printf("%s, map 2: %d to %d\n", Thread.currentThread(), i, i + 10);
                return i + 10;
            })
            .subscribe(new BaseSubscriber<Integer>() {
                @Override
                protected void hookOnSubscribe(Subscription subscription) {
                    System.out.println(Thread.currentThread() + " hookOnSubscribe");
                    requestUnbounded();
                }

                @Override
                protected void hookOnNext(Integer value) {
                    System.out.println(Thread.currentThread() + " hookOnNext: " + value);
                }

                @Override
                protected void hookOnComplete() {
                    System.out.println(Thread.currentThread() + " hookOnComplete");
                    latch.countDown();
                }
            });

        latch.await();
    }
```

publishOn() 메서드를 이용하면 next, complete, error신호를 별도 쓰레드로 처리할 수 있다.
map(), flatMap() 등의 변환도 publishOn()이 지정한 쓰레드를 이용해서 처리한다.


### subscribeOn Operator 이용한 구독 처리 쓰레드 스케줄링
subscribeOn은 구독이 발생한 직후 실행될 스레드를 지정하는 오퍼레이터이다.

subscribeOn()을 사용하면 Subscriber가 시퀀스에 대한 request신호를 별도 스케줄러로 처리합니다.(시퀀스를 실행할 스케줄러를 지정한다)

subscribeOn()으로 지정한 스케줄러는 시퀀스의 request 요청 처리뿐만 아니라 첫 번째 publishOn() 지정 이전까지의 신호 처리를 실행한다.

```java
    @Test
    void subscribeOnTest() throws InterruptedException
    {
        CountDownLatch latch = new CountDownLatch(1);

        Flux.range(1, 6)
            .log() // 보다 상세한 로그 출력 위함
            .subscribeOn(Schedulers.boundedElastic())
            .map(i -> {
                System.out.printf("%s, map 1: %d to %d\n", Thread.currentThread(), i, i + 10);
                return i + 10;
            })
            .subscribe(new BaseSubscriber<Integer>() {
                @Override
                protected void hookOnSubscribe(Subscription subscription) {
                    System.out.println(Thread.currentThread() + " hookOnSubscribe"); // main thread
                    request(1);
                }

                @Override
                protected void hookOnNext(Integer value) {
                    System.out.println(Thread.currentThread() + " hookOnNext: " + value); // SUB 쓰레드
                    request(1);
                }

                @Override
                protected void hookOnComplete() {
                    System.out.println(Thread.currentThread() + " hookOnComplete"); // SUB 쓰레드
                    latch.countDown();
                }
            });

        latch.await();
    }
```

따라서 위 코드를 실행하면 Flux.range()가 생성한 시퀀스의 신호 발생뿐만 아니라 map() 실행, Subscriber의 next, complete 신호 처리를 "SUB" 스케줄러가 실행한다.

참고로 시퀀스의 request 요청과 관련된 로그를 보기 위해 log() 메서드를 사용할 수 있다.

subscribeOn과 publishOn을 적절히 혼합하면 데이터를 emit하는 스레드와 emit된 데이터를 가공 처리하는 스레드를 적절하게 분리할 수 있다.

### parallel()
subscribeOn과 publishOn은 동시성을 가지는 논리적 스레드에 해당하지만 parallel()은 병렬성을 가지는 물리적 스레드에 해당한다.

parallel()은 라운드로빈 방식으로 CPU 논리적 코어(물리적 스레드) 개수 만큼 스레드를 병렬로 실행한다.

예를 들어 4코어 8스레드 CPU라면 8스레드를 병렬로 실행한다.


# Context
Context란 어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보를 의미한다.

Reactor에서 Context는 Operator같은 Reactor 구성요소 간에 전파되는 키/밸류 형태의 저장소라고 정의한다.
여기서 전파는 다운스트림에서 업스트림으로 컨텍스트가 전파되어 Operator 체인상의 각 Operator가 해당 Context의 정보를 동일하게 이용할 수 있음을 의미한다.

Reactor의 Context는 Subscriber와 매핑됩니다. 즉 구독이 발생할 때마다 해당 구독과 연결된 하나의 컨텍스트가 생긴다라고 볼 수 있다.

Context는 Operator 체인의 아래에서부터 위로 전파한다. 따라서 Operator 체인 상에서 Context read 메서드가 Context write 메서드 밑에 있을 경우에는 write된 값을 read할 수 없다.

Context를 이용하기 적합한 경우는 API 인증정보 같은 독립성을 가지는 정보를 전송하는 경우에 유용하다고 볼 수 있다.

### Context에 쓰기
ContextWrite() Operator를 통해 컨텍스트에 데이터를 쓸 수 있다.
실제로 데이터를 쓰는 동작은 Context API 중 하나인 put()을 통해 쓸 수 있다.

### Context에서 읽기
deferContextual() 오퍼레이터를 통해 원본 데이터 소스 레벨에서 읽을 수 있다.
deferContextual()은 defer() Operator와 같은 원리로 동작하는데 Context에 저장된 데이터와 원본 데이터 소스의 처리를 지연시키는 역할을 한다.

Operator 체인의 중간에서 Context의 데이터를 읽기 위해서는 transformDeferredContextual() 오퍼레이터를 사용한다.

Reactor에서는 Operator 체인 상의 서로 다른 스레드들이 Context의 저장된 데이터에 손쉽게 접근할 수 있다.
그래서 Context.put()을 통해 컨텍스트에 데이터를 쓴 후에 매번 불변 객체를 다음 contextWrite() Operator로 전달함으로써 스레드 안전성을 보장한다.

### 자주 사용되는 컨텍스트 API
* put(k,v)
  key/value 형태로 컨텍스트에 쓴다.

* of(k1,v1,k2,v2..)
  여러 개의 값을 컨텍스트에 쓴다.

* putAll(contextView)
  현재 컨텍스트와 파라미터로 입력된 컨텍스트뷰를 머지한다.

* delete(k)
  키에 해당하는 밸류를 제거한다.
