---
layout: default
title: Reactor Operator
description: 
grand_parent: Java
parent: Reactive Streams
nav_order: 3
---

# Reactor Operator
{: .no_toc}

# Table of contents
{: .no_toc .text-delta }

1. TOC 
{:toc}

# Flux

### just()
sequence를 생성하는 가장 쉬운 방법. just() 메서드는 시퀀스로 사용할 데이터가 이미 존재할 때 사용한다.  

```java

```

### range(), generate()
11부터 15까지의 Integer를 생성하는 코드
```java
Flux<Integer> seq = Flux.range(11, 5);
```


##### Flux<T> generate(Consumer<SynchronousSink<T>> generator)

generate()를 사용하면 함수를 이용해서 생성이 가능하다. 동기 방식으로 한 번에 1개의 데이터를 생성할 때 사용한다. generator는 Subscriber로부터 요청이 왔을 때 신호를 생성한다. generate()가 생성한 Flux는 다음과 같은 방식으로 신호를 발생한다.

1. Subscriber의 요청에 대해 인자로 전달받은 generator를 실행한다. generator를 실행할 때 인자로 SynchronousSink를 전달한다.
2. generator는 전달받은 SynchronousSink를 사용해서 next, complete, error 신호를 발생한다. 한 번에 1개의 next() 신호만 발생할 수 있다.


```java
Consumer<SynchronousSink<Integer>> randGen = new Consumer<>() {
    private int emitCount = 0;
    private Random rand = new Random();
 
    @Override
    public void accept(SynchronousSink<Integer> sink) {
        emitCount++;
        int data = rand.nextInt(100) + 1; // 1~100 사이 임의 정수
        logger.info("Generator sink next " + data);
        sink.next(data); // 임의 정수 데이터 발생
        if (emitCount == 10) { // 10개 데이터를 발생했으면
            logger.info("Generator sink complete");
            sink.complete(); // 완료 신호 발생
        }
    }
};
 
Flux<Integer> seq = Flux.generate(randGen);
 
seq.subscribe(new BaseSubscriber<>() {
    private int receiveCount = 0;
    @Override
    protected void hookOnSubscribe(Subscription subscription) {
        logger.info("Subscriber#onSubscribe");
        logger.info("Subscriber request first 3 items");
        request(3);
    }
 
    @Override
    protected void hookOnNext(Integer value) {
        logger.info("Subscriber#onNext: " + value);
        receiveCount++;
        if (receiveCount % 3 == 0) {
            logger.info("Subscriber request next 3 items");
            request(3);
        }
    }
 
    @Override
    protected void hookOnComplete() {
        logger.info("Subscriber#onComplete");
    }
});
```
randGen의 accept() 메서드는 1~100 사이의 임의 정수를 생성한 뒤 인자로 SynchronousSink의 next() 메서드를 이용해서 next 신호를 발생한다. emitCount가 10이면(즉 데이터를 10개 발생했다면) SynchronousSink#complete() 메서드를 이용해서 complete 신호를 발생한다.
 
randGen은 신호 발생 기능을 제공할 뿐이며 실제 시퀀스는 Flux.generate()를 이용해서 생성했다.

seq.subscribe() 메서드에 전달한 Subscriber는 구독 시점에 3개의 데이터를 요청하고(hookOnSubscribe() 메서드의 request(3) 코드), 데이터를 3개 수신할 때마다 다시 3개의 데이터를 요청한다(hookOnNext() 메서드의 request(3) 코드).
 
콘솔에 관련 문장을 출력해서 실행 흐름을 알 수 있도록 했다. 위 코드를 실제로 실행해보면 다음 내용이 콘솔에 출력된다. 원래 출력에는 빈 줄이 없는데 쉬운 구분을 위해 빈 줄을 넣었고 Subscriber의 출력은 으로 표시했다.  <= Subscriber

```
Subscriber#onSubscribe              <= Subscriber
Subscriber request first 3 items    <= Subscriber
 
Generator sink next 17
Subscriber#onNext: 17               <= Subscriber
Generator sink next 83
Subscriber#onNext: 83               <= Subscriber
Generator sink next 53
Subscriber#onNext: 53               <= Subscriber   
Subscriber request next 3 items     <= Subscriber
 
Generator sink next 12
Subscriber#onNext: 12               <= Subscriber
Generator sink next 38
Subscriber#onNext: 38               <= Subscriber
Generator sink next 90
Subscriber#onNext: 90               <= Subscriber
Subscriber request next 3 items     <= Subscriber
 
Generator sink next 23
Subscriber#onNext: 23               <= Subscriber
Generator sink next 70
Subscriber#onNext: 70               <= Subscriber
Generator sink next 76
Subscriber#onNext: 76               <= Subscriber
Subscriber request next 3 items     <= Subscriber   
 
Generator sink next 52
Subscriber#onNext: 52               <= Subscriber
Generator sink complete
Subscriber#onComplete               <= Subscriber
```

##### Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator)
##### Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator, Consumer<? super S> stateConsumer)

stateSupplier는 값을 생성할 때 사용할 최초 상태이다. BiFunction 타입의 generator는 인자로 상태와 SynchronousSink를 입력받아 결과로 다음 상태를 리턴하는 함수이다. 앞서 예제와 마찬가지로 SynchronousSink를 사용해서 신호를 생성한다. 두 번째 generate() 메서드의 stateConsumer는 상태를 정리할 때 사용한다. generator가 complete 신호나 error 신호를 발생하면 상태 정리를 위해 stateConsumer를 실행한다.

다음은 상태를 사용하는 Flux.generate()를 이용해서 임의 숫자 10개를 발생시키는 Flux를 생성하는 코드 예이다.  

```java
Flux<String> flux = Flux.generate(
        () -> { // Callable<S> stateSupplier
            return 0;
        },
        (state, sink) -> { // BiFunction<S, SynchronousSink<T>, S> generator
            sink.next("3 x " + state + " = " + 3 * state);
            if (state == 10) {
                sink.complete();
            }
            return state + 1;
        });
```
이 코드에서 flux는 "3 x 0 = 0" 부터 "3 x 10 = 30" 까지의 데이터를 담은 next 신호를 차례대로 발생하고 state 값이 10이 되면 compelete 신호를 발생한다.
 


Flux.just()나 Flux.generate()는 데이터 생성 과정 자체가 동기 방식이다. Subscriber로부터 데이터 요청이 오면 그 시점에 SynchronousSink를 이용해서 데이터를 생성한다. 반면에 별도 쓰레드를 이용해서 비동기로 데이터를 생성해야 하는 경우에는 SynchronousSink를 사용할 수 없다. 게다가 SynchronousSink는 한 번에 하나의 next 신호만 발생할 수 있다. 예를 들어 아래 코드는 에러를 발생한다.

```java
Flux<String> flux = Flux.generate(
        () -> 1,
        (state, sink) -> {
            sink.next("Q: 3 * " + state);
            sink.next("A: " + (3 * state)); // 에러!
            if (state == 10) {
                sink.complete();
            }
            return state + 1;
        });
```

데이터 자체를 비동기로 생성해야 하거나 한번에 다수의 next 신호를 발생해야 할 경우 Flux.generate()로는 처리할 수 없다. 이 때에는 Flux.create() 메서드를 사용해야 한다.


# create()
Flux.generate()는 Subscriber로부터 요청이 있을 때에 next 신호를 발생하는 Flux를 생성한다. 즉 pull 방식의 Flux를 생성한다. 이는 단순하지만 데이터 발생을 비동기나 push 방식으로 할 수 없다는 제약도 있다. Flux.create()를 사용하면 이런 제약 없이 비동기나 push 방식으로 데이터를 발생할 수 있다.

##### **Flux.create()를 이용한 pull 방식 메시지 생성**

Flux.create() 메서드를 이용해서 pull 방식으로 메시지를 생성하는 예제 코드이다.

```java
Flux<Integer> flux = Flux.create( (FluxSink<Integer> sink) -> {
    sink.onRequest(request -> { // request는 Subscriber가 요청한 데이터 개수
        for (int i = 1; i <= request; i++) {
            sink.next(i); // Flux.generate()의 경우와 달리 한 번에 한 개 이상의 next() 신호 발생 가능
        }
    });
});
```

Flux.create() 메서드의 파라머티는 함수형 타입 Consumer<? super FluxSink<T>>이다. 이 Consumer는 FluxSink를 이용해서 Subscriber에 신호를 발생할 수 있다.

FluxSink#onRequest(LongConsumer) 메서드의 Consumer는 Subscriber가 데이터를 요청했을 때 불린다. 이때 LongConsumer는 Subscriber가 요청한 데이터 개수를 전달받는다. 위 코드에서는 클라이언트가 요청한 데이터 개수만큼 next 신호를 발생하고 있다.

Flux.generate()와의 차이점은 Flux.generate()의 경우 한 번에 한 개의 next 신호만 발생할 수 있었던 데 비해 Flux.create()는 한 번에 한 개 이상의 next() 신호를 발생할 수 있다는 점이다.

##### **Flux.create()를 이용한 push 방식 메시지  생성**

Flux.create()를 이용하면 Subscriber의 요청과 상관없이 비동기로 데이터를 발생할 수 있다. 다음 코드를 보자.

```java
DataPump pump = new DataPump();

Flux<Integer> bridge = Flux.create((FluxSink<Integer> sink) -> {
    pump.setListener(new DataListener<Integer>() {
        @Override
        public void onData(List<Integer> chunk) {
            chunk.forEach(s -> {
                sink.next(s); // Subscriber의 요청에 상관없이 신호 발생
            });
        }

        @Override
        public void complete() {
            logger.info("complete");
            sink.complete();
        }
    });
});
```

이 코드에서 DataPump는 데이터를 어딘가에서 데이터가 오면 setListener()로 등록한 DataListener의 onData()를 실행한다고 가정하자. DataListener#onData() 메서드는 FluxSink#next()를 이용해서 데이터를 발생한다. DataListener#onData() 메서드는 Subscriber의 데이터 요청과 상관없이 호출된다. 즉 위 코드는 Subscriber의 요청과 상관없이 데이터를 push한다.

##### Flux.create()와 배압
Subscriber로부터 요청이 왔을 때(FluxSink#onRequest) 데이터를 전송하거나(pull 방식) Subscriber의 요청에 상관없이 데이터를 전송하거나(push 방식) 두 방식 모두 Subscriber가 요청한 개수보다 더 많은 데이터를 발생할 수 있다. 예를 들어 아래 코드를 보자.

```java
Flux<Integer> flux = Flux.create( (FluxSink<Integer> sink) -> {
    sink.onRequest(request -> {
        for (int i = 1; i <= request + 3 ; i++) { // Subscriber가 요청한 것보다 3개 더 발생
            sink.next(i);
        }
    });
});
```

이 코드는 Subscriber가 요청한 개수보다 3개 데이터를 더 발생한다. 이 경우 어떻게 될까? 기본적으로 Flux.create()로 생성한 Flux는 초과로 발생한 데이터를 버퍼에 보관한다. 버퍼에 보관된 데이터는 다음에 Subscriber가 데이터를 요청할 때 전달된다.



요청보다 발생한 데이터가 많을 때 선택할 수 있는 처리 방식은 다음과 같다.

* IGNORE : Subscriber의 요청 무시하고 발생(Subscriber의 큐가 다 차면 IllegalStateException 발생)
ERROR : 익셉션(IllegalStateException) 발생
* DROP : Subscriber가 데이터를 받을 준비가 안 되어 있으면 데이터 발생 누락
* LATEST : 마지막 신호만 Subscriber에 전달
* BUFFER : 버퍼에 저장했다가 Subscriber 요청시 전달. 버퍼 제한이 없으므로 OutOfMemoryError 발생 가능
Flux.create()의 두 번째 인자로 처리 방식을 전달하면 된다.

```java
Flux.create(sink -> { ... }, FluxSink.OverflowStrategy.IGNORE);
```

### Flux.fromStream(), Flux.fromIterable()을 이용한 Flux 생성

```
Stream<String> straem = Files.lines(Paths.get(filePath));
Flux<String> seq = Flux.fromStream(straem);
seq.subscribe(System.out::println);
```

Flux.fromIterable()을 이용하면 Iterable을 이용해서 Flux를 생성할 수 있다. List나 Set과 같은 콜렉션에서 Flux를 생성하고 싶을 때 이 메서드를 사용하면 된다.



### flatMap()
* https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMap-java.util.function.Function-
### concatMap()  
* https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#concatMap-java.util.function.Function-

### flatMapSequential
* https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#flatMapSequential-java.util.function.Function-

### Tips for flatMap, concatMap, flatMapSequential

use flatMap for fast parallel calls (max 256 limit — default concurrency), when ordering of elements is not required.

use concatMap for sequential calls when strict ordering of elements is required.

use flatMapSequential for parallel calls with strict ordering of elements.

### share()
share() 오퍼레이터는 원본 Flux를 멀티캐스트하는 새로운 Flux를 리턴하는 오퍼레이터이다.

즉 여러 Subscriber가 하나의 Flux를 공유하도록 합니다. 하나의 원본 Flux를 공유하여 다 같이 사용하기 때문에 어떤 Subscriber가 이 원본 Flux를 먼저 구독해버리면 데이터 emit을 시작하게 되고, 이후에 다른 Subscriber가 구독하는 시점에는 이미 emit된 데이터는 전달받을 수 없게된다.

우리는 share() 오퍼레이터를 통해 Cold Sequence를 Hot Sequence로 동작하게 할 수 있다.

```java
    @Test
    @DisplayName("share() Operator를 이용해 Cold Sequence를 Hot Sequence로 동작하게 할 수 있다")
    void testHotSequence() throws InterruptedException {
        String[] singers = {"로이킴", "뉴진스", "트와이스", "테일러 스위프트"};

        // 4명의 가수는 2초 간격으로 무대에 나와 노래를 부를 예정입니다.
        Flux<String> concertFlux = Flux.fromArray(singers)
                .delayElements(Duration.ofSeconds(2))
                .share();

		// 첫 관객 관람(구독)을 하여 공연이 시작됐습니다.
        concertFlux.subscribe(singer -> System.out.printf("[%s] 관객 A가 %s의 무대를 보았습니다.%n", Thread.currentThread(), singer));

        Thread.sleep(2500); //공연 시작 2.5초 뒤.. 관객 B가 관람(구독) 시작
        concertFlux.subscribe(singer -> System.out.printf("[%s] 관객 B가 %s의 무대를 보았습니다.%n", Thread.currentThread(), singer));

        Thread.sleep(6000);
    }
```

```
[Thread[parallel-1,5,main]] 관객 A가 로이킴의 무대를 보았습니다.
[Thread[parallel-2,5,main]] 관객 A가 뉴진스의 무대를 보았습니다.
[Thread[parallel-2,5,main]] 관객 B가 뉴진스의 무대를 보았습니다.
[Thread[parallel-3,5,main]] 관객 A가 트와이스의 무대를 보았습니다.
[Thread[parallel-3,5,main]] 관객 B가 트와이스의 무대를 보았습니다.
[Thread[parallel-4,5,main]] 관객 A가 테일러 스위프트의 무대를 보았습니다.
[Thread[parallel-4,5,main]] 관객 B가 테일러 스위프트의 무대를 보았습니다.
```



# Mono

### just()
sequence를 생성하는 가장 쉬운 방법. just() 메서드는 시퀀스로 사용할 데이터가 이미 존재할 때 사용한다. Flux의 just()와 다른점은 입력이 하나뿐이라는 것이다.

Mono.just(null)로 null을 주면 NullPointerException이 발생한다. 데이터를 발생하지 않는 Mono를 생성하고 싶으면 Mono.empty()를 사용한다. 값이 있을 수도 있고 없을 수도 있는 Mono를 생성할 때는 justOrEmpty()를 사용하면 된다.

```java
// null을 값으로 받으면 값이 없는 Mono
Mono<Integer> seq1 = Mono.justOrEmpty(null); // complete 신호
Mono<Integer> seq2 = Mono.justOrEmpty(1); // next(1) 신호- complete 신호

// Optional을 값으로 받음
Mono<Integer> seq3 = Mono.justOrEmpty(Optional.empty()); // complete 신호
Mono<Integer> seq4 = Mono.justOrEmpty(Optional.of(1)); // next(1) 신호 - complete 신호
```

### cache()

cache() 오퍼레이터는 Cold Sequence로 동작하는 Mono를 Hot Sequence로 변경해주고, emit된 데이터를 캐시한 뒤 구독이 발생할 때마다 캐시된 데이터를 전달해준다.

```java
@Test
    void testCacheHotSequence() throws InterruptedException {
        var mono = Mono.fromCallable(() -> {
                    System.out.println("Go!");
                    return 5;
                })
                .map(i -> {
                    System.out.println("Double!");
                    return i * 2;
                });

        var cached = mono.cache();

        System.out.println("Using cached"); // Hot Sequence로 동작한다.
        System.out.println("1. " + cached.block()); // 최초 한 번 연산
        System.out.println("2. " + cached.block()); // 캐싱 반환
        System.out.println("3. " + cached.block()); // 캐싱 반환

        System.out.println("Using NOT cached"); // Cold Sequence로 동작한다.
        System.out.println("1. " + mono.block()); // 처음부터 다시 동작
        System.out.println("2. " + mono.block()); // 처음부터 다시 동작
        System.out.println("3. " + mono.block()); // 처음부터 다시 동작
    }
```

```
Using cached
Go!
Double!
1. 10
2. 10
3. 10
Using NOT cached
Go!
Double!
1. 10
Go!
Double!
2. 10
Go!
Double!
3. 10
```

테스트 결과를 보면 cache()사용 시 Hot Sequence로 동작하여 호출마다 처음부터 돌지 않고, 2번째 호출부터는 캐시된 값을 반환한다.
