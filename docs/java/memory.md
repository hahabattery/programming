---
layout: default
title: Memory
parent: Java
nav_order: 10
---



# 다양한 팁

 * JMXTERM 사용방법
   * https://linux.systemv.pe.kr/jmxterm-%ec%bb%a4%eb%a7%a8%eb%93%9c-%eb%9d%bc%ec%9d%b8-jmx/

 * JMX 로 TOMCAT 모니터링 하기.
   * https://linux.systemv.pe.kr/jmx-%eb%a1%9c-tomcat-%eb%aa%a8%eb%8b%88%ed%84%b0%eb%a7%81-%ed%95%98%ea%b8%b0/

 * jconsole
   * jconsole 실행하고 pid를 선택하면 연결됨.
   * C:\Users\songw\AppData\Local\Temp\hsperfdata_songw 디렉토리에 "everyone"에 권한을 할당해줘야 함.

 * JSTACK 과 THREAD ID, CPU, MEM 사용량 체크
   * https://linux.systemv.pe.kr/jstack-%ea%b3%bc-thread-id-cpu-mem-%ec%82%ac%ec%9a%a9%eb%9f%89-%ec%b2%b4%ed%81%ac/

# Java default option 알아내기

 * 옵션들이 없다면 Java 는 기본적으로 어떤 값을 사용하는 지 확인하는 
```
./java -XX:+PrintCommandLineFlags -version
```


### Memory Management

![](/images/java/memory/memory-1.png)
![](/images/java/memory/memory-2.png)
int와 같은 primitives는 stack에 저장됨
object는 heap에 저장되고, referece는 stack에 저장됨.

![](/images/java/memory/memory-3.png)

![](/images/java/memory/memory-4.png)
printLinst()를 호출하면, argument가 stack에 저장되고, myList는 접근이 되지 못한다.
![](/images/java/memory/memory-5.png)
printList()가 리턴되면, printList()함수 호출시 생성된 stack정보는 날라간다.


![](/images/java/memory/memory-6.png)
java에는 함수 호출시에 인자로 reference를 전달하는 feature가 없기 때문에, 무조건 passinb by value가 일어난다.
object의 경우에는 stack에 저장된 object의 reference가 전달됨.

![](/images/java/memory/memory-7.png)
renameCustomer() 호출시에 c라는 Customer 객체의 reference 변수가 stack할당되고,
호출된 함수에서 String 객체를 변경한다. 호출된 함수에는 c변수는 접근할 수 없다.
Java에서 String은 immutable이기 때문에, 변경운 안되고, 새로 할당되게 되고, 기존 String객체는 GC된다.
![](/images/java/memory/memory-8.png)
함수가 리턴되면, 다시 c 변수에 접근이 가능해짐.

- - -

 * final variable cannot be changed
 * final method cannot be overriden by subclassed
 * final classes cannot be subclass, so can not inherit from final class.
 * 만약에 상속받는 클래스가 있고, 그 클래스 안에 final method가 있으면, final method를 override할 수 없음.
 * final 키워드를 붙이면, java compiler가 최적화(inlining)할 가능성이 커지기 때문에, 최적화 측면에서 권장됨.
 
 * 정확히 말하면, final variable은 바뀔 수 없는 것이 아니라, 한번만 값을 할당할 수 있는 것을 의미한다.
 * java에는 const 키워드가 없고, final과 static 키워드를 이용한다.



![](/images/java/memory/memory-9.png)
![](/images/java/memory/memory-10.png)
 * final keyword는 object를 가리키는 stack변수의 값을 변경하지 못하는 것이지, 실제 heap상의 object를 변경 못하는 것은 아님.
 * java에서는 heap상의 object를 변경하는 것을 막을 수단이 존재하지 않음. 타언어에서는 const키워드를 이용해 구현되기도 했지만, java에서는 구현되지 않음.

# JVM

JVM의 대표적인 기능들은 ...

 * 맞춤 메모리 관리(Adaptive memory management)
 * 가비지 컬렉션(garbage collection)
 * just-in-time compilation
 * 동적 클래스로딩(dynamic classloading)
 * 락 최적화(lock optimization)
 
실행 시점에서, JVM은 지속적인 측정과 프로파일링을 기반으로 애플리케이션이나 그것의 일부를 핸들링하는 방법을 최적화한다.
JVM ergonomic 에 의해서 JVM의 타입은 JVM이 시작될때에 운영체제와 활용할수 있는 하드웨어를 고려한 기준에의해서 자동적으로 선택되어진다.

 * JVM Type이 결정되어지는 기준
   * https://docs.oracle.com/javase/6/docs/technotes/guides/vm/server-class.html



# JVM option



##### JVM option 출력하기(전체)

```java
java -XX:+PrintFlagsFinal
```

##### client JVM option 출력하기

java -client -XX:+PrintFlagsFinal Benchmark

```
  [Global flags]
uintx AdaptivePermSizeWeight               = 20               {product}
uintx AdaptiveSizeDecrementScaleFactor     = 4                {product}
uintx AdaptiveSizeMajorGCDecayTimeScale    = 10               {product}
uintx AdaptiveSizePausePolicy              = 0                {product}
[...]
uintx YoungGenerationSizeSupplementDecay   = 8                {product}
uintx YoungPLABSize                        = 4096             {product}
 bool ZeroTLAB                             = false            {product}
 intx hashCode                             = 0                {product}
```


##### server JVM option 출력하기

java -server -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal Benchmark | grep ":"
```
uintx InitialHeapSize                     := 57505088         {product}
uintx MaxHeapSize                         := 920649728        {product}
uintx ParallelGCThreads                   := 4                {product}
 bool PrintFlagsFinal                     := true             {product}
 bool UseParallelGC                       := true             {product}
```




 * “=“는 네번째 플래그의 값이 기본값(Default Value)이라는 걸 의미
 * ”:=“는 사용자 혹은 JVM 인체공학에 의해서 지정된 값


 * -XX:+PrintFlagsInitial 는 "=" (기본값)항목들만 보여준다.
 * -XX:+PrintFlagsFinal 는 최종 적용된 항목들을 모두 보여준다.

 * -XX:+PrintCommandLineFlags 는 ":=" 항목들만 보여준다.


# JIT Compilation
![](/images/java/memory/java-compile-1.png)
![](/images/java/memory/java-compile-2.png)

##### JVM flags

 * 표준 플래그(stand flag)
   * -server와 같은 것
 * -X 플래그
 * -XX 플래그 문법(syntax)에 대해 한마디. 모든 XX 플래그들은 “-XX” 로 시작하지만 문법이 의존하는 플래그 타입이 다르다.
   * 불린 플래그(boolean flag)에서, “+“나 ”-” 둘다 가지며 플래그를 지정하기 위해서 JVM 옵션의 실제 이름만 가진다. 예를들어, -XX:+<name> 은 옵션 <name> 을 활성화하고 -XX:-<name> 은 이 옵션을 비활성화 한다.
   * 텍스트 문자열처럼 불린(boolean)이 아닌 값을 가지거나 정수를 가지는 플래그들에서, “=” 에 따라오는 플래그 이름을 가지고 값을 지정한다. 예를들어, -XX:<name>=<value> 는 옵션 <name> 에 값 <value>을 지정한다.

##### VM 32bit or 64bit
![](/images/java/memory/client-option-1.png)
![](/images/java/memory/jvm-version-1.png)


##### -server 와 -client

 * -d64는 64bit server compiler
 * 클라이언트 VM은 오직 32bit 시스템에서만 활용가능함.
 * 미리 정의된 JVM이 불만이라면, 우리는 서버와 클라이언트 VM 사용을 규정하기 위해 -server 와 -client 플래그를 사용할 수 있다. 
 * - XX:-TieredCompilation
   * make the JVM to run in interpreted mode only
   * 매우 적은 라인(한두 줄?)의 코드만으로 만들어진 어플리케이션에 유용할 듯 하지만, 보통 사용될 일은 없다.



##### -version 과 -showversion

```
$ java -version
java version "1.6.0_24"
Java(TM) SE Runtime Environment (build 1.6.0_24-b07)
Java HotSpot(TM) Client VM (build 19.1-b02, mixed mode, sharing)
```

 * 출력된 내용을 보면 JAVA 버전 넘버(1.6.0_24)와 사용된 정확한 JRE의 빌드ID(1.6.0_24-b07) 를 보여준다. 또, 우리는 이름을(HotSpot) 볼수 있고, JVM의 빌드ID(19.1-b02) 와 타입(Client)도 볼 수 있다. 
 * 거기에 더해, 우리는 JVM 이 믹스드 모드(mixed mode)로 동작한다는걸 알 수 있다. 이 실행 모드는 기본적인 HotSpot 모드이고 실행 타임에 동적으로 바이트 코드(byte code)를 네이티브 코드(nate code)로 컴파일 한다는 걸 의미한다. 
 * 또, 우리는 클래스 데이터 공유(class data sharing)가 활성화 되었다는 것도 알 수 있다. 클래스 데이터 공유는 모든 JAVA 프로세스들이 클래스로더에 의해서 자원을 공유하는데 사용되어지는 읽기전용 캐쉬에 JRE 의 시스템 클래스들을 저장하는 기법이다. 클래스 데이터 공유는 매번 jar archive들로부터 모든 클래스 데이터를 읽어들이는 것과 비교해볼때 성능면에서 대체로 이득이 있다.
 * -version 플래그는 위 데이터를 출력한 후에 즉각 JVM을 종료한다. 
 * 그러나, 같은 출력결과를 만드는데 사용되어질 수 있는 -showversion 는 유사한 플래그지만 주어진 자바 애플리케이션을 실행하고 처리한다.


##### -Xint, -Xcomp, 그리고 Xmixed

 * -Xint 는 JVM에게 모든 바이트코드(Bytecode)를, 통상적으로 10배 이상 아주 느려지는 것이 수반되는, 인터프리터 모드로 실행하도록 강제한다.
 * -Xcomp 는 명시적으로 정반대로 동작하도록 강제하는데 그것은 JVM이 처음 사용시에 모든 바이트코드를 네이티브코드(Native code)로 컴파일하는데 결국 최고의 최적화 레벨을 적용하게 된다.
 * mixed mode가 기본값
 * Xcomp 세팅은 JVM에게 JIT 컴파일러(JIT Compiler)가 효율적으로 네이티브 코드를 만들어내는 것을 방해하게 한다. JIT 컴파일러는 실행타임에 메소드 사용 프로파일들을 생성한 다음에 실제 애플리케이션 동작을 위해 차례대로 그들의 일부나 혹은 추론을 해서 싱글 메소드들을 최적화한다. 이러한 최적화 테크닉의 일부들은, 예를들어 optimistic branch prediction, 맨 처음에 애플리케이션의 프로파일링 없이 효율적으로 적용되어질 수 없다. 또 다른 관점으로 메소드는 그들 스스로가 애플리케이션에서 어떤 종류의 지점을 구성하는데 연관되어 있다는 것이 증명되었을때 전체가 컴파일되어 진다. 오직 한번이나 아주 적게 호출되어지는 메소드들은 인터프리터 모드로 실행되는 것을 지속하게 되고 따라서 compilation 과 최적화(optimzation) 비용을 절약하게 된다.
 
 

 


- - -

##### -XX:+PrintCompilation

![](/images/java/memory/java-compile-3.png)

 * 1st col the number of milliseconds since the virtual machine started.
   * 이 시점에 컴파일을 시작했다는 의미.
 * 2nd col the orfer in which the method or code block was compiled. 코드간의 의존성에 의해서 컴파일됨.
 * 3nd col N = Native Mehod, S = Syncronized Method, % = code cached, 
 * 4nd col 0 = just interpreted, 1-4 = progressivley deeper level of compilation.

##### -XX:CICompilerCount=n
 * compiler 개수 설정
 
```
java -XX:CICompilerCount=6 -XX:+PrintCompilation Main 15000
```

##### -XX:CompileThreshold=n
 * number of method invocations/branches before (re-)compiling


##### -XX:+CITime

```
$ java -server -XX:+CITime Benchmark
[...]
Accumulated compiler times (for compiled methods only)
------------------------------------------------
  Total compilation time   :  0.178 s
    Standard compilation   :  0.129 s, Average : 0.004
    On stack replacement   :  0.049 s, Average : 0.024
[...] 
```
 * (29 컴파일러 태스크를 위해) 총 0.178 초를 소비했다. 물론, “on stack replacement” 를 가지는 0.049초는 스택에서 현재 메소드의 컴파일 시간이다. 이 기술은 성능 기준에 맞는 유형을 구현하기위해서 단순하지 않지만 실제로 아주 중요하다. “on stack replacement” 없이 오랜 실행 시간을 가지는 메소드들은 그들의 컴파일된 카운터파트(counterpart, 짝 혹은 또 다른 부분 )로 즉각 교체되어질 수 없다.
 * 다시말하면, 클라이언트 VM 와 서버 VM 과 비교는 흥미롭다. 클라이언트 VM에 해당하는 통계는 비록 55개 메소드를 컴파일했다는 것을 나타냈지만 이들을 컴파일하는데 총 0.021초만 소비했다. 따라서, 서버 VM 은 클라이언트 VM보다 적게 컴파일을 했지만 시간은 더 많이 소비했다. 이러한 행동의 이유는 서버 VM은 네이티브 코드를 생성할때에 좀 더 최적화를 수행하기 때문이다.


```
$ java -server -Xcomp -XX:+CITime Benchmark
[...]
Accumulated compiler times (for compiled methods only)
------------------------------------------------
  Total compilation time   :  1.567 s
    Standard compilation   :  1.567 s, Average : 0.002
    On stack replacement   :  0.000 s, Average : -1.#IO
[...]
```
 * -Xcomp 를 사용해 컴파일하는데 소비한 시간 1.567 초는 기본 세팅 값(ex, mixed mode) 보다 약 10배나 많은 시간을 소비했다. 여전히, 애플리케이션은 mixed 모드보다 더 느리게 동작한다. 클라이언트 VM 은 -Xcomp 를 사용했을 경우 726개의 메소드들을 컴파일하는데 0.208 초를 소비했다. 이것은 -Xcomp 를 사용한 서버 VM보다 더 느린 것이다. (역, 말이 이상하다. server VM 에서는 993 라인에 1.567 초를 client VM 에서는 726 라인에 0.208 초를 소비했는데 어째서 server VM 보다 느리다고 한 걸까?)


##### -XX:+UnlockExperimentalVMOptions
 * 어떤 플래그들은 실제로 자바 애플리케이션에서 사용하지 않고 오직 JVM 개발을 위해서 사용되도록 해놨다. 만일 플래그가 -XX:+UnlockExperimentalVMOptions 로도 활성화가 되지 않지만 어떤 이유에선지 그 플래그를 반드시 필요로한다면, 여러분은 JVM 의 디버그 빌드를 가지는 행운을 누릴 수 있다

- - -

##### -XX:+UnlockDiagnosticVMOptions -XX:+LogCompilation

 * 로그에 compile 정보가 저장됨.
 * c2 compiler로 컴파일 된 것인지 여부를 확인 가능.




##### -XX:+PrintOptoAssembly
 * JVM 은 바이트코드 컴파일로 생성된 네이티브 코드 결과들을 살펴볼 수 있도록 해준다. -XX:+PrintOptoAssembly 플래그로 인해서 컴파일러 스레드에 의해 생성 된 네이티브 코드는 표준 출력과 “hotspot.log” 파일에 모두 기록됩니다. 이 플래그를 사용은 서버 VM의 디버그 빌드를 실행하도록 요구한다. 우리는, 죽은 코드 제거하기처럼, JVM이 실제 어떤 종류의 최적화를 수행하는지를 이해하기 위해서 -XX:+PrintOptoAssembly 의 출력을 연구할 수 있다.

- - -

##### -XX:+PrintCodeCache
![](/images/java/memory/java-compile-4.png)

 * code cache의 사용 용량이 출력됨
 * code cache의 사이즈 설정은 java 버전에 따라 다름.
   * JVM이 64bit인 경우 48MB, 32bit인 경우 32MB


 * Option
   * InitialCodeCacheSize
     * 프로그램이 시작할 시점의 code cache size, 컴퓨터의 가용 메모리를 참고해서 설정되는데, 자주 160kb로 설정됨.
   * ReservedCodeCacheSize
     * maximum size of the code cache.
   * CodeCacheExpansionSize
     * how quickly the code cache should grow.


```
java -XX:ReservedCodeCacheSize=28m -XX:+PrintCodeCache Main
```




# Heap Tuning

##### -Xms and -Xmx (or: -XX:InitialHeapSize and -XX:MaxHeapSize)

 * 초기 힙 크기를 128 megabytes 와 최대 힙 크기를 2 gigabytes 로 세팅
```
java -Xms128m -Xmx2g MyApp
```

##### -XX:+HeapDumpOnOutOfMemoryError and -XX:HeapDumpPath

 * OutOfMemoryError 발생했을때에 자동적으로 힙 점프를 생성
 * 디스크용량을 고려해서 -XX:HeapDumpPath 파일 저장위치를 지정
 
 * 덤프를 생성하고 스크립트 실행까지 하는 사용예
```
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/heapdump.hprof -XX:OnOutOfMemoryError ="sh ~/cleanup.sh" MyApp
```

##### -XX:PermSize and -XX:MaxPermSize
 * -XX:MaxPermSize 는 permanent 세대의 최대 크기를 지정하는 것이고 반면에 -XX:PermSize 는 JVM 시작시에 초기화되는 값
 * permanent 세대 크기는 -XX:MaxHeapSize 로 지정되어지는 힙 크기의 일부로 계산되지 않는다. 
 
 * 사용예
```
java -XX:PermSize=128m -XX:MaxPermSize=256m MyApp
```

##### -XX:InitialCodeCacheSize and -XX:ReservedCodeCacheSize

 * 만약 코드 캐쉬 메모리를 모두 사용하게 되면, JVM 은 경고 메시지를 출력하고 “interpreted-only mode” 로 전환한다. JIT 컴파일러는 정지되고 더이상 바이트코드(bytecode)는 네이티브 코드로 컴파일 될지 않을 것이다



##### -XX:+UseCodeCacheFlushing

 * 만약 코드 캐쉬가 지속적으로 증가한다면, 예를들어 hot deployment 로 인해서 발생된 메모리 릭으로 인해서, 코드 캐쉬를 늘리는 것은 단지 필연적으로 발생될 오버플로우(OverFlow)를 지연시킬 뿐이다. 오버플로우를 비하기 위해, 코드 캐쉬 메모리가 가득찼을때에 JVM이 몇몇 컴파일된 코드를 정리하도록 시키기위해 우리는 재미있고 비교적 새로운 옵션을 시도할 수 있다. 우리는 -XX:+UseCodeCacheFlushing 플래그를 지정하면 된다. 
 
 
 
 
# Garbage Collection

![](/images/java/memory/young_gc.png)

 * 객체는 보통 “Eden” 에서 태어나고 각 young generation GC 때마다 servivor 공간들 사이를 왔다갔다 한다. 만약 객체들이 많은 young generation GC를 통과할때까지 생존한다면, 그것은 최종적으로 old generation 으로 옮겨지고 다른 살아있는 객체들과함께 머물게 된다. 마지막으로 객체가 old generation 에서 죽었을때, 아주 무거운 GC 알고리즘중에 하나때문에 좀 더 많은 노력으로 그것들을 수집하게된다.
 * young generation 크기가 왜 중요한지 확실해졌다. 만약 young generation 이 아주 작으면, 짧게 생존하는 객체들은 그들이 수집되기 좀 더 힘든 old generation 으로 빠르게 옮겨진다. 반대로 young generatin이 아주 크면, 우리는 어쨌거나 나중에 old generation 으로 옮겨지게될 오랫동안 생존한 객들을에대해서 불필요하게 아주 많은 복사가 이루어진다. 따라서 우리는 작은 young generation 크기 와 큰 young generation 크기 사이에 접점을 찾을 필요가 있다.

##### -XX:NewSize and -XX:MaxNewSize

 * young generation 의 크기의 양을 명시적으로 하한과 상한으로 지정 
 * when setting -XX:MaxNewSize we need to take into account that the young generation is only one part of the heap and that the larger we choose its size the smaller the old generation will be. 이것은 old generation 보다 you gerneration 크기를 좀 더 크게 선택하지 말아야 하는데, 모든 객체들을 young generation 에서 old generation 으로 이동하기 위해 GC가 필요할 경우에 최악의 상황이 된다. 따라서 -Xmx/2 는 -XX:MaxNewSize 에 대한 상한이다.
 * 성능상의 이유로 우리는 -XX:NewSize 플래그를 사용함으로써 young generation 의 초기 크기를 지정할 수 있다. 우리가 어린 개체가 할당되는 속도를 알고 천천히 시간이 지남에 따라 그 크기로 젊은 세대의 성장에 필요한 비용의 일부를 절약 할 수 있는 경우에 유용하다.

##### -XX:NewRatio
 
 * old generation 의 상대적인 비율로 young generation 크기를 지정
 * -XX:NewRatio=3 일때 old generation 는 young generation 보다 3배 더 크게 된다. old generation 은 힙의 3/4 을 young generation 은 힙의 1/4 를 얻게 된다.
 
```
java -XX:NewSize=32m -XX:MaxNewSize=512m -XX:NewRatio=3 MyApp
``` 

 * 위의 예에서는 절대치와 상대치가 둘다 나왔기 때문에, 절대치가 우선시되어서 young generation 이 결코 32MB 이하나 512MB 이상을 초과하지 않게된다.
 
 * 절대 또는 상대적인 young generation 크기가 존재한다면 바람직한 일반적인 룰은 없다. 만약 애플리케이션의 메모리 사용량을 잘 알고 있다면, 전체 힙과 young generation 양쪽 모두 고정된 크기로 지정하는 것이 유리할 수 있고 이것은 비율을 지정하는데도 유용할 수 있다. 만약 우리가 조금만 알거나 애플리케이션에 대해서 전혀 아는게 없다면, 이러한 접근은 단순하게 JVM 을 동작하게하고 플래그 주변을 지저분하게 하지 않게 한다. 만약 애플리케이션이 부드럽게 동작한다면, 우리는 아무것도 필요하지 않은 경우 여분의 노력을 하지 않아도된다는 것에 행복해질 수 있다. 그리고 우리는 성능 문제나 OutOfMemoryErrors가 발생하면 우리는 여전히 튜닝을 하기전에 문제의 근본 원인을 좁힐 수 있는 의미있는 일련의 측정이 필요하다.
 

##### -XX:SurvivorRatio

 * 플래그 -XX:SurvivorRatio 는 -XX:NewRatio 유사하지만 young generation 내부 영역에 적용된다. -XX:SurvivorRatio 의 값은 두 survivor 공간중 하나에 상대적으로 얼마나 크게 Eden 을 정할건지를 지정한다. 예를들어, -XX:SurvivorRatio=10는 “To”에 비해서 “Eden”이 10배 큰 치수를 가리킨다. (그리고 같은 시점에서 “From”에 비해 10배 크게). 결과적으로, “Eden”은 young generation 에서 10/12 를 가지는 반면에 “To”와 “From”은 각각 1/12 를 가진다. 주목할 것은 두개의 survivor 공간은 늘 같은 크기다.
 * survivor 공간의 크기가 주는 영향은 무엇일까? survivor 공간은 “Eden”과 비교해서 아주 작다고 가정한다. 그리고 우리는 “Eden”에 새롭게 할당된 객체를 아주 많이 가지고 있다고 생각해보자. 만약 이러한 모든 객체들이 다음번 GC 동안 모두 수집되어질 수 있다면, “Ede”은 다시 비워지고 모든것은 괜찮아진다.하지만, 만약 이러한 young 객체들중에 몇몇이 여전히 참조되어지는 상태라면, 우리는 survivor 공간에 그들을 수용하기 위해 아주 작은 공간을 가진다. 결론적으로, 이러한 대부분의 객체들은(아직도 참조되어지고 있는 객체들) 첫번째 GC 이후에 old generation 으로 옮겨질 겁니다. 이제 반대 상황을 생각해보자. survivor 공간의 크기가 상대적으로 크다고 가정해보자. 그리고 그들은 자신들의 목적을 수행하기 위해서 많은 공간을 가지고 있다면 객체를 수용하기위해서 하나 이상의 GC들을 발생시키지만 여전히 young 객체는 죽는다. 그러나, 아주 작은 “Eden” 공간은 좀 더 빠르게 고갈될 것이고, 실행되는 young generation GC들의 양은 증가한다.
 * 요약하면, 우리는 너무 빠르게 old generation 으로 이동되어지는 짧은 삶을 사는 객체들의 숫자를 최소화하길 원한다. 하지만 우리는 또 young generation GC 시간과 발생 숫자를 최소화하길 원한다. 다시 말해서 우리는 우리들 스스로 애플리케이션의 특성에 따라서 타협점을 찾을 필요가 있다. 알맞은 타협점을 찾기위한 좋은 시작점은 특정 애플리케이션에 객체들의 연령 분포에 대해서 아는 것이다.

##### -XX:+PrintTenuringDistribution
 * young generation GC 의 survivor 공간에 포함된 모든 객체들의 연령 분포(age distribution)를 출력

```
Desired survivor size 75497472 bytes, new threshold 15 (max 15)
- age   1:   19321624 bytes,   19321624 total
- age   2:      79376 bytes,   19401000 total
- age   3:    2904256 bytes,   22305256 total
```

 * 첫번째 라인은 “To” survivor 공간의 타켓 사용률(target utilization)이 약 75MB 임을 말해준다. 또, old generation 으로 이동되지 전에 객체가 young generation 에 머무는 GC의 수를 나타내는 “임계 보유기간”에 대한 정보도 보여준다. (i.e., 이것이 추진되기 전에 객체의 최대 수명) 이 예제에서, 우리는 현재 “임계 보유기간”은 15이고 최대값도 15임을 알 수 있다.
 * 다음 라인은, “임계 보유기간” 보다 훨씬 어린 각각의 객체, 현재 그 나이를 가지는 모든 객체의 총 바이트 수를 보여준다 (만약 객체가 현재 특정 연령에 대해 존재하지 않는 경우, 그 라인은 생략된다). 예를들어, 약 19MB 는 한번 GC에서 살아남은 양, 약 79KB는 두번 GC로 살아남은 양, 약 3MB는 세번째 GC에서 살아남은양이다. 각 라인에 끝에는, 그 연령(age)에 올라온 모든 객체들의 총 양(byte) 이다. 따라서 , 마지막 라인에 “Total” 값은 “To” survivor 공간에 현재 약 22MB 의 객체 데이터를 포함한다는 것을 가리킨다. “To”의 타켓 사용률(target utilization)은 75MB 이고 현재 “임계 보유기간”은 15, 우리는 현재 young generation GC 로인해서 old generation 으로 승진시켜야할 객체가 없다는 결론을 내릴 수 있다. 이제 다음 GC가 다음과 같은 출력을 이끌어내는 것을 가정해보자.
 
```
Desired survivor size 75497472 bytes, new threshold 2 (max 15)
- age   1:   68407384 bytes,   68407384 total
- age   2:   12494576 bytes,   80901960 total
- age   3:      79376 bytes,   80981336 total
- age   4:    2904256 bytes,   83885592 total
```

 * 이전에 보유기간 분포의 출력과 비교해보자. 이전 출력에서 2, 3세대(age) 의 모든 객체들은 여전히 “To” 에 있다. 왜냐하면 우리는 3, 4세대에 대해 같은 양이 출력된것을 정확하게 볼 수 있다.(역, 뭔 소린지… total 값이 비슷하다는 건지) 그래서 우리는 GC로 인해서 “To” 에 있는 특정 객체들이 성공적으로 수집되었다고 결론을 내릴 수 있다. 왜냐하면 현재 우리는 2 세대의 객체가 12MB만 가지고 있는 반면에 이전 출력에서 1세대는 19MB를 가졌었기 때문이다. 마지막으로 우리는 약 68MB의 새로운 객체들은, 1세대 부분에서, 마지막 GC 동안 “Eden” 에서 “To”로 이동되었다는 것을 알 수 있다.
 * 주목할 것은 “To” 에 총 bytes 수는 – 여기서는 거의 84MB – desired number of 75MB 보다 크다. 결론적으로 JVM은 보유기간 임계값(tenuring threshold)을 15에서 2로 줄였고, 그래서 다음 GC에 특정 객체들은 “To”로 강제로 이주될 것이다. 이러한 객체들은 만약 특정시간에 죽게되면 수집되어지거나 여전히 참조되고 있다면 old generation 으로 이동되어진다


##### -XX:InitialTenuringThreshold, -XX:MaxTenuringThreshold and -XX:TargetSurvivorRatio
 * -XX:+PrintTenuringDistribution 으로 조절하는데 필요한 값을 출력해볼 수 있음.
 
 * -XX:InitialTenuringThreshold 과 -XX:MaxTenuringThreshold 는 보유기간 임계값의 초기값과 최대값을 지정
 * -XX:MaxTenuringThreshold=10 -XX:TargetSurvivorRatio=90 조합은 보유기간 임계값에 대해 10 의 상한과 “To” survivor 공간에 대해 90% 의 타켓 사용률을 지정

 * 사용팁
   * 만약 최종적으로 최대 보유기간 임계값에 도달하기전 보유기간 임계값 많은 객체들은 오래됐고 늙었다걸 보여주고 있다면 이것은 -XX:MaxTenuringThreshold 는 아주 크다는 걸 말해준다.
   * -XX:MaxTenuringThreshold 의 값이 1보다 크지만 대부분의 객체가 1보다 큰 세대에 도달하지 못할거라면 우리는 “To” 타켓 사용률(the target utilization)을 찾아봐야 한다. Should the target utilization never be reached, then we know that all young objects get collected by the GC, which is exactly what we want. 그러나, 만약 타켓 사용률이 빠르게 도달한다면, 적어도 1세대 이후에 혹은 조기에 특정 객체들이 old generation 으로 이동되어진다. 이 경우에, 우리는 타켓 사용률과 그들이 크기를 증가시킴으로써 survivor 공간을 개선시도할 수 있다.



##### -XX:+NeverTenure 과 -XX:+AlwaysTenure
 * -XX:+NeverTenure를 지정하면, 객체들은 결코 old generation 으로 옮겨지지 않는다. 이 동작은 old generation 이 필요가 없다고 확신할때 가능하다. 그러나, 그러한 플래그는 분명히 위험하고 예약한 힙 메모리의 절반을 낭비하게 한다.
 * -XX:+AlwaysTenure 는 작동할 수 있다. 첫번째 GC 에서 모든 young 객체들은 즉각 old generation 으로 옮겨 survivor 공간을 사용하지 않게 한다. 이 플래그의 적절한 사용 케이스를 찾기는 어렵다. 테스트 환경에서 어떤 일이 발생할지를 찾기위해서 재미로 할 수 있다 (LoL )




 * GC 쓰레드들은 어떤 객체들이 아직도 참조되어 있는지 아닌지를 결정하기위해서 노력한다. 이러한 이유 때문에, 애플리케이션 쓰레드들은 반드시 GC 동안 정지된다
 * 최대 처리율이나 최소 일시 정지시간은 상충관계이기 때문에, GC 알고리즘을 사용하거나 디자인할때에 우리는 목적이 무엇인지를 결정해야 한다. GC 알고리즘 두 목적중에 하나만(최대 처리율이나 최소 일시 정지시간에 맞춰) 타켓으로하거나 그들사이에 타협점을 찾기위해 노력
 

### throughput-oriented GC

##### -XX:+UseSerialGC
 * 단일 프로세서 코어만 사용하는 JVM에서만 권장됨


##### -XX:+UseParallelGC
 * 다중GC 쓰레드를 사용하기 위한 옵션

##### -XX:+UseParallelOldGC
 *  “old”는 실제로 old generation 을 가르치는 용어
 * 높은 처리율이 필요하고 JVM이 두개 이상의 다중 코어를 사용할때라면 이 플래그 사용을 추천됨

##### -XX:ParallelGCThreads
 * 다중 GC를 위해 사용되어질 GC 스레드의 수를 지정
 * 이 플래그를 명시적으로 지정하지 않으면, JVM은 활용가능한 프로세서들의 수를 기반으로 계산한 기본값을 사용한다. 이것을(기본 값) 결정하는 요인은 Runtime.availableProcessors() 자바 메소드에 의해서 리턴된 N 값
 * N>8 활용가능한 프로세서들이 있다면 GC 쓰레드의 수는 3+5N/8으로 조정됨
 * 예를들어, 4개 서버 JVM이 16개 프로세서 코어를 가진같은 머신에서 동작중이라면, -XX:ParallelGCThreads=4 는 다른 JVM 의 GC와 상호간섭이 일어나지 않기 위해서 적절한 선택이다.


##### -XX:GCTimeRatio
 * -XX:GCTimeRatio=N 값은 애플리케이션 쓰레드의 실행시간에(총 프로그램 실행 시간과 연관된) 대해 N/(N+1)의 목표비율을 지정
 * 예를들어 -XX:GCTimeRatio=9 우리는 애플리케이션 쓰레드들이 적어도 총 실행시간의 9/10 을 활성화하게 요구할 수 있다. (그리고, 따라서, 나머지 1/10 이 GC 쓰레드들이다.) 


##### -XX:MaxGCPauseMillis
 * -XX:MaxGCPauseMillis=<value>는 JVM에게 최대 일시정지 시간을 목표값(밀리초)으로 설정
 * 만일 최대 정지시간과 최소 처리율에 대한 목표값을 지정하면, 최대 일시 정지 시간 목표를 달성하는 것에 우선순위를 더 높게 가진다. 물론, JVM이 열심히 노력한다고 하더라도 양쪽 모두 목표 달성을 보증하지는 않는다. 결국, everything depends on the behavior of the application at hand
 * 최대 일시 정지 시간 목표를 설정했을때, 우리는 너무 작은 값을 선택하지 않도록 주의해야 한다. 우리는 지금까지 알고 있듯이, 일시정지 시간을 낮게 유지하기 위해, JVM은 달성할수있는 처리량에 심각하게 영향을 주는 총 GC 갯수를 증가시켰다. 그것은 애플리케이션이 주요 목표처럼, 마치 웹 애플리케이션의 경우처럼, 낮은 일시정시간을 요구하기 때문에 나는 처리율 컬렉터만 사용하길 추천하지 않고 대신 CMS 컬렉터로 바꾸길 권한다.


##### -XX:-UseAdaptiveSizePolicy
 * JVM이 GC 성능을 향상시키기위한 근거가 있는 경우에는 자동으로 설정값을 변경하는데, 이것을 비활성화하기 위한 옵션이다.





### CMS Collector

 * 처리량 컬렉터는 늘 애플리케이션 쓰레드들을 잠시 멈추게하지만, 아마도 적지않은 시간, 그 시간을 애플리케이션이 안전하게 무시할수 있도록 한다. 
 * 그와 대조적으로, CMS 컬렉터는 대부분 애플리케이션 쓰레드들과 동시적으로 실행되도록 디자인되었고 아주 적은(혹은 짧은) 잠시 정지시간만 발생시킨다.
 * 힙 파편화는 발생 가능한데, 처리율 컬렉터와 다르게, CMS 컬렉터는 단편화(defragmentation)를 위한 어떤 메커니즘도 포함하지 않는다. 
 * 이러한 일이 발생하면, 동시성 알고리즘은 더이상 도움이 안되며 따라서, 최후의 수단으로, JVM은 전체 GC(Full Gc)를 발생시킨다. 전체 GC는 처리량 수집기에 의해 사용 된 알고리즘을 실행하는 것을 상기하면 파편화 이슈는 해결된다. – 그것은 또한 애플리케이션을 정지시킨다
 * 두번째 도전은 애플리케이션의 높은 객체 할당 비율이다. 만약 객체 인스턴스화하는 비율이 컬렉터가 힙에서 죽은 객체를 제거하는 비율보다 아주 높다면 동시적 알고리즘은 여러번 실패한다. 어떤점에서, old generation은 young generation 에서 승격된 객체를 수용할 충분히 활용가능한 공간이 없을 것이다. 이러한 상황은 “concurrent mode failure” 라고하며 JVM은 힙 파편화때와 마찬가지로 움직인다. 바로 전체 GC.
 * 이러한 시나로중에 하나가 실제로 일어날때를보면 (공교롭게도 실제 프로덕트 시스템에서도 일반적으로 발생한다), old generation 에 불필요한 엄청난 양의 객체들이 그곳에 존재한다는 것이 판명된다. 한가지 가능한 대책은 old generation 으로 짧게 사는 객체들의 조기 승격을 방지하기 위해 young generation 을 증가시키는 것이다. 다른 해결책은 프로파일러(profiler)을 사용하거나 운영중인 시스템의 힙 덤프를 얻어 과도한 객체할당에 대해서 객체들을 규명하고 애플리케이션을 분석해서 최종적으로는 많은 양의 객체할당을 줄이는 것이다.

##### -XX:+UseConcMarkSweepGC
 *  CMS 컬렉터를 활성화하기 위해 필요하다. 기본적으로, HotSpot 은 처리율 컬렉터를 대신 사용한다.
 
##### -XX:+UseParNewGC
 * CMS 컬렉터를 사용할때, 이 플래그는 다중 쓰레드를 사용해 young generation GC의 parallel 실행을 활성화한다
 * -XX:+UseConcMarkSweepGC를 지정하면 자동적으로 -XX:+UseParNewGC가 활성화 된다. 결과적으로, 만일 parallel young generation GC가 매력적이지 않다면 -XX:-UseParNewGC 세팅함으로써 비활성화할 필요가 있다.

##### -XX:+CMSConcurrentMTEnabled
 * 이 플래그를 지정하면, 동시적 CMS 단계는 다중 쓰레드로 동작한다. 따라서 다중 GC 쓰레드는 모든 애플리케이션 쓰레들에서 parallel에서 작동한다. 


##### -XX:ConcGCThreads
 * -XX:ConcGCThreads=<value> 플래그는 사용할 쓰레드 개수를 정의한다.
 * 예를들어, 값이 4면 CMS 주기의 모든 동시적 단계는 4개의 쓰레드를 사용해 동작한다는 뜻이다. 비록 높은 쓰레드의 개수가 동시적 CMS 단계의 속도를 높여줄수 있지만 추가적으로 동기화 오버헤드를 발생시킨다. 따라서 특정 애플리케이션을 다룰때, CMS 쓰레드의 수의 증가가 실제로 성능향상을 가지고 오는지 그렇지 않는지를 측정해야만 한다.
 * 명시적으로 지정해주지 않으면 JVM은 처리량 컬렉터에서 알려진 -XX: ParallelGCThreads 플래그 값에 기반에 기본값 parallel CMS 쓰레드 수를 계산한다. 사용된 계산 공식은 ConcGCThreads = (ParallelGCThreads + 3)/4 이다.


##### -XX:CMSInitiatingOccupancyFraction
 * 처리량 컬렉터는 힙이 꽉 찼을때만 GC 주기를 시작한다. 예를들어, 새로운 객체를 할당할 공간이 없거나 객체를 old generation으로 승격시킬 공간이 없을때. CMS 컬렉터에서는 동시적 GC 동안에도 애플리케이션이 동작중여야하기 때문에 오랜시간을 기다리면 안된다. 따라서, 애플리케이션이 out of memory 되기전에 GC 주기를 끝내기 위해서 CMS 컬렉터는 처리량 컬렉터보다 일찍 GC 주기를 시작할 필요가 있다.
 * -XX:CMSInitiatingOccupancyFraction=<value> 통해서 퍼센트로 old generation 힙 공간의 사용량으로 표시되어 지정될 수 있다. 예를들어 값이 75면 old generation의 75%를 획득했을때에 첫번째 CMS 주기를 시작하라는 의미다. 전통적으로 CMSInitiatingOccupancyFraction 의 기본값은 68 이다. (오랫동안 경험을 통해 얻은 수치다)

##### -XX+UseCMSInitiatingOccupancyOnly
 * 런타임 통계에 기반해 CMS 주기를 시작할지 결정하지 않도록 JVM 에게 지시하기 위해 -XX+UseCMSInitiatingOccupancyOnly 를 사용할 수 있다. 대신, 이 플래그가 활성화된 때에, JVM은 첫음 한번만이 아닌 매번 CMS주기에서 CMSInitiatingOccupancyFraction 값을 사용한다. 
 * 그러나, 대부분의 경우 JVM이 우리 인간보다 GC 의사결정을 좀 더 잘한다는 것을 염두에 둬야 한다. 


##### -XX:+CMSClassUnloadingEnabled
 * 처리량 컬렉터와 비교해, CMS 컬렉터는 기본적으로 permanent generation 에 GC를 수행하지 않는다. 만약 permanent generation GC가 필요하다면, -XX:+CMSClassUnloadingEnabled 통해서 활성화될 수 있다
 * 이 플래그를 지정하지 않는다하더라도, 공간이 부족하게 되면 permanent generation 의 가비지 컬렉트를 시도할 것이지만 컬렉션은 동시적으로 수행되지 않을 것이다. – 대신, 다시 한번, 전체 GC가 동작할 것이다.

##### -XX:+CMSIncrementalMode
 * 점진적 모드(incremental mode)를 활성화 한다. 
 * 점진적 모드는 애플리케이션 쓰레드에 완전히 양도(yield)되도록 동시적 단계를 주기적으로 잠시 멈추게 한다. 결론적으로, 컬렉터는 전체 CMS 주기를 완료시키기 위해서 아주 오랜시간을 가질 것이다. 따라서, 점진적 모드 사용은 일반적인 CMS 사이클에서 동작중인 컬렉터 쓰레드가 애플리케이션 쓰레드와 아주 많이 간섭이 발생되고 있다고 측정되어질경우에 유효하다. 
 * 이것은 동시적 GC 수용을 위해 충분한 프로세스를 활용하는 현대 서버 하드웨어에서 아주 드물게 발생된다.


##### -XX:+ExplicitGCInvokesConcurrent 와 -XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses
 * System.gc() 호출에 의해서 명시적으로 호출되는  예기치못한 시스템GC의 stop-to-world에 대해서 우리 스스로 보호할 수 있다.
 * -XX:+ExplicitGCInvokesConcurrent 플래그는 JVM이 시스템 GC 요청이 있을때마다 전체GC 대신 CMS GC를 실행하도록 지시한다. 
 * -XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses 는 시스템 GC가 요청될 경우에 추가로 permanent generation 을 CMSGC에 포함하도록 해준다.



##### -XX:+DisableExplicitGC
 * 어떤 타입의 컬렉터를 사용하던지간에 시스템 GC 요청을 완벽하게 무시하도록 JVM에게 지시



### GC Logging

##### -XX:+PrintGC
 * -XX:+PrintGC 나 혹은 별명인 -verbose:gc 는 모든 young generation GC와 모든 풀GC에 대해 로깅
```
[GC 246656K->243120K(376320K), 0,0929090 secs]
[Full GC 243120K->241951K(629760K), 1,5589690 secs]
```

 * 첫째라인에서, 246656K→243120K(376320K) 는 GC가 점유 힙 메로리를 246656K 에서 243120K 로 줄였다는 뜻이다. GC 당시에 힙 용량은 376320K 이고 GC 는 0.0929090 시간을 소비했다.
 * 단순한 gC 로깅 포캣은 사용되는 GC 알고리즘과 아무런 상관이 없고 더이상 상세함을 제공하지 않는다. 위의 예제에서 우리는 GC가 young 에서 old generation 으로 객체가 이동될때에 로그에 기록할 수 없다. 그러한 이유로, 상세한 GC 로깅은 단순한 로깅보다 훨씬 유용하다.


##### -XX:+PrintGCDetails

```
[GC
    [PSYoungGen: 142816K->10752K(142848K)] 246648K->243136K(375296K),
    0,0935090 secs
]
[Times: user=0,55 sys=0,10, real=0,09 secs]
```

 * 우리는 단순한 GC 로그로부터 몇가지 요소를 알수 있었다. 점유 힙 메모리를 246648K 에서 243136K 감소시키고 0.0935090 초를 소비한 young generation GC가 있다. 추가로, young generation 자체에 대한 정보를 얻는데, 이것의 용량과 점유률뿐만아니라 사용된 컬렉터가 무엇인지를 알 수 있다. 위 예제에서, “PSYoungGen” 컬렉터는 점유된 young generation 힙 메모리가 142816K 에서 10752K 로 줄일 수 있었다.

 * 우리는 young generation 용량을 알고 있기 때문에, young generation 이 다른 객체할당을 수용할 수 없기때문에 GC를 발생시켰다라고 쉽게 이야기할 수 있다. 활용가능한 142848K 중에 142816K 이 사용되었다. 게다가, 우리는 우리는 young generation에서 제거 된 개체의 대부분이 아직 살아 있고 old generation 으로 이동되었다고 결론 내릴 수 있다: 142816K→10752K(142848K) 와 246648K→243136K(375296K) 를 비교해 young generation 이 거의 완벽하게 비였음에도 불구하고, 전체 힙 점유률은 거의 같게 남았다.

 * 상세 로그 섹션의 “Times”은 GC에 의해서 사용된 운영체제의 커널 공간(“sys”), 사용자 공간(“user”)으로 나뉜 CPU 시간에 대한 정보를 포함한다. 또, GC가 동작중에 소비한 real time(“real”) 가 보인다. (0.09 sms 로그에서 보여준 0.0935090 시간의 반올림일 뿐이다.) 만약 위 예제처럼 CPU time 이 소비된 real time 보다 상당히 높다면 GC가 다중 쓰레드를 사용해 동작했다고 결론내릴 수 있다. 이러한 경우, 기록된 CPU time 은 모든 GC 쓰레드들의 CPU time의 총 합이다. 그리고 추가로, 나는 위 예제에서 컬렉터가 8개의 쓰레드를 사용했다고 말할 수 있다.


##### 
```
[Full GC
    [PSYoungGen: 10752K->9707K(142848K)]
    [ParOldGen: 232384K->232244K(485888K)] 243136K->241951K(628736K)
    [PSPermGen: 3162K->3161K(21504K)],
    1,5265450 secs
]
[Times: user=10,96 sys=0,06, real=1,53 secs]
```

 * young generation 에 대한 추가된 상세함으로 인해서 old 와 permanert generation 에 대해 상세함을 우리에게 제공해준다. 전체 세가지 세대들에 대해, 우리는 사용된 컬렉터, GC 전후에 점유율, GC 시에 용량을 알 수 있다. 주목할 것은 총 힙에 대해 보여준 각 숫자는 ( 243136K→241951K(628736K) ) 는 young 과 old generation 의 숫자 합과 같다. 위 예제에서, 241951K 의 총 힙을 점유했고 9707K 는 young generation, 232244K 는 old generation 을 점유했다. 풀GC는 1.53초 소비됐고 user space에서 CPU time 10.96 초는 GC가 다중 쓰레드로 상되었음을 보여준다.(위와같이 8개 쓰레드)

 * 다른 세대들에대한 상대한 출력은 GC 발생에대한 추론을 가능하게 한다. 만약에, 모든 세대에 대해, GC전에 점유한 로그 상태가 거의 현재 용량과 같다면 이 세대는 GC가 발생가능성이 높다. 하지만, 위 예제에서, 이것은 세가지 세대에대해 해당되지 않는다. 그러면 무엇이 이 경우에 GC를 발생시켰을까? 처리율 컬렉터에서 GC 인체공학이 세가지 세대중에 하나가 고갈되기전에 GC가 이미 실행되어야 한다고 결정했다면 실제로 이러한 일이 발생될 수 있다.

 * 풀GC 또한 애플리케이션이나 외부 JVM 인터페이스중에 하나를 통해서 명시적으로 요청이 있을때에 발생될 수도 있다. “system GC” 경우에는 GC 로그에서 “Full GC” 대신에 “Full GC(System)”으로 시작되는 라인을 가지기 때문에 쉽게 파악할 수 있다.

 * 시리얼 컬렉터(Serial Collector) 에서, 상세 GC 로그는 처리량 컬렉터와 매우 유사하다. 한가지 다른 점이라면 다른 GC 알고리즘을 사용하고 있기 때문에 다양한 섹션들에 이름들이 다르다.(예를들어, old generation 섹션은 “ParOldGen” 대신에 “Tenured” 라 부른다). 정확한 이름의 컬렉션을 사용하는 것이 좋은데, JVM의 의해서 사용되어지는 특정 가비지 컬렉션의 로그로부터 추론을 할 수 있기 때문이다.

 * CMS 컬렉터(CMS Collector)에서, young generation GC에 상세 로그는 역시 처리량 컬렉션과 매우 유사하지만 old generation GC에 대해서는 같지 않다. CMS 컬렉터에서 old generation GC들은 다른 단계를 사용해 애플리케이션에 동시적으로 작동한다. 그렇기 때문에, 그것의 출력은 풀GC 출력과 다르다. 추가적으로 다른 단계에대한 라인들은 일반적으로 동시적 컬렉션이 동작중에 발생된 young generation GC 에대한 라인에 의해서 로그와 분리된다. 다른 컬렉터들을 이미 봐왔기 때문에 GC 로그의 모든 요소들이 익숙하고 그래서 다른 단계에 대한 로그를 이해하기란 어렵지 않다. 다만 처리중일때에 특별히 조심하고 애플리케이션과 동시적으로 모든 단계가 실행된다는 것을 상기해야만 한다. 따라서, 반대되는 stop-the-world 컬렉션들처럼, 개별적인 단계 혹은 전체 GC 주기가 오랫동안 실행되는 시간이 항상 문제가 있다는 것은 아니다.


```
[GC 
	[1 CMS-initial-mark: 5506743K(7340032K)] 5535371K(8283776K), 
	0.0092830 secs
] 
[Times: user=0.01 sys=0.00, real=0.02 secs] 
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.112/0.112 secs] [Times: user=0.43 sys=0.05, real=0.11 secs] 
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.026/0.029 secs] [Times: user=0.08 sys=0.01, real=0.03 secs] 
[CMS-concurrent-abortable-preclean-start]
[CMS-concurrent-abortable-preclean: 1.300/1.339 secs] [Times: user=2.73 sys=0.48, real=1.34 secs] 
[GC
	[YG occupancy: 450411 K (943744 K)]
	3506541.032: [Rescan (parallel) , 0.0495870 secs]
	3506541.082: [weak refs processing, 0.0663520 secs] 
	[1 CMS-remark: 5506743K(7340032K)] 5957154K(8283776K), 
	0.1211210 secs
] 
[Times: user=0.46 sys=0.00, real=0.12 secs] 
[CMS-concurrent-sweep-start]
[CMS-concurrent-sweep: 0.288/0.288 secs] [Times: user=0.92 sys=0.18, real=0.29 secs] 
[CMS-concurrent-reset-start]
[CMS-concurrent-reset: 0.017/0.017 secs] [Times: user=0.03 sys=0.01, real=0.02 secs]
```

 * part 7에서 알아봤듯이, 풀GC들은 CMS 컬렉터가 재시간에 CMS 주기가 완료되지 않으면 발생 될 수 있다. 만약 그게 발생하면, GC 로그는 추가적으로 풀GC 발생 원인이 무엇인지와 같은 힌트를 포함한다. 예를들어, “concurrent mode failure” 와 같은.


##### -XX:+PrintGCTimeStamps and -XX:+PrintGCDateStamps
 * 이것은 (단순하게 혹은 상세하게) GC 로그에 시간과 날짜 정보를 추가하는게 가능
 * -XX:+PrintGCDateStamps 를 지정하면 로그가 쓰여질때에 각 시작 라인이 절대 날짜와 시간

##### -Xloggc
 * -Xloggc:<file> 로 출력할 파일 지정 가능
 
##### “Manageable” Flags

 * 관리 플래그들을 지정하기 위해서 우리는 JDK에 포함된 jinfo 를 사용하거나 JMX 클라이언트와 HotSpotDiagnostic MXBean 의 연산인 setVMOption를 호출해서, 실행중인 JVM에서 우리가 원할때마다 언제든지 GC 로깅을 활성화하거나 비활성화할 수 있다. 







