---
layout: default
title: Debugging
parent: Java
nav_order: 10
---

운영중에 발생하는 여러 이슈를 대응하기 위한 정보들.


# 환경변수

 * System.getProperty("변수명"); <= 이코드 정상 동작하기 위해서는  
   * jar실행시에 -D 로 지정하던가
   * 이클립스 내에서는 Run Configuration -> Arguments에 -DVAR_NAME="BLAH"  추가하면 됨.
   * Tomcat 의 경우,  톰캣/conf/catalina.properties 에 추가 (또는 /var/lib/tomcat8/conf/catalina.properties)



# Memory

##### -Xms
 * -Xms<size> : Java Heap의 최초 크기(Start Size)를 지정한다. Java Heap은 -Xms 옵션으로 지정한 크기로 시작

##### -Xmx
 * -Xmx<size> : 최대 Java Heap Size, JVM이 최대 가질 수 있는 메모리이다.


##### -XX:PermSize
 * -XX:PermSize=<size> : 클래스
   * Permanent Generation의 최초 크기를 지정한다. Permanent Generation의 최대 크기는 MaxPermSize옵션에 의해 지정된다. 
   * 많은 수의 Class를 로딩하는 Application은 크기의 Permanent Generation을 필요로 하며, 
   * Permanent Generation의 크기가 작아서 Class를 로딩하지 못하면 Out of Memory Error가 발생한다.

##### -XX:MaxPermSize
 * -XX:MaxPermSize=<size> : Permanent Generation의 최대 크기를 지정한다. Permanent Generation의 시작 크기는 PermSize옵션에 의해 지정된다. 
 * 많은 수의 Class를 로딩하는 Application은 PermSize와 MaxPermSize옵션을 이용해 Permanent Generation의 크기를 크게 해주는 것이 좋다. 
 * Permanent Generation의 크기가 작을 경우에는 Out of Memory Error가 발생한다.

##### -XX:NewSize
 * -XX:NewSize<size> : Young Generation의 최소 size(시작크기). 객체가 생성되어 저장되는 초기공간의 Size로 Eden+Survivor 영역

##### -XX:MaxNewSize
 * -XX:MaxNewSize<size> : Young Generation의 최대 size. Young Generation의 크기는 NewSize와 MaxNewSize옵션에 의해 결정된다.

##### -XX:NewRatio=<Value>
 * Young Generation과 Old Generation의 영역 비율
 * 2로 설정할 경우 Young:Old = 1:2가 된다. 전체 힙중 Young Generation의 크기는 33%가 된다.
 
##### -XX:SurvivorRatio=<Value>
 * Survivor Space와 Eden Space의 영역 비율
 * Survivor 영역은 1,2 총 두개가 존재한다. 이 값을 6으로 설정하면 s1:s2:eden = 1:1:6이라는 의미이다.
 * Survivor 영역이 크면, Tenured Generation으로 옮겨가기 전 중간 버퍼의 영역이 커진다. Full GC의 빈도를 줄이는 역할을 할 수 있다. Eden Space 크기가 줄어들어 Minor GC가 자주 발생한다.

#####  주의사항
 * MaxPermSize는 -Xmx 로 지정한 메모리 용량과 별도로 할당된다. 
 * -Xmx가 256m 이고, -XX:MaxPermSize가 256m 이라면, 최대 512m이 할당될 수 있다는 것이다.
 ```
 JAVA_OPTS="-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms3072m -Xmx3072m -XX:NewSize=384m 
 ```


##### 세팅예

 * Xms, Xmx를 동일하게 세팅하는 예
 ```
 -XX:MaxNewSize=384m -XX:PermSize=256m -XX:MaxPermSize=256m -XX:+DisableExplicitGC"
 ```

 * 이유
   * - Xms로 init 메모리를 잡고, committed 도달할 때까지 Used용량이 점차 증가하는데, 
   * committed에 도달시 메모리 추가할당시 시스템 부하발생 (WAS가 몇 ms가량 멈출 가능성 있음)
   * 메모리 용량은 init < used < committed < max 
   * 보통 운영시스템에서 Xms와 Xmx를 동일하게 지정하는 이유는 init와 max사이에서 used 메모리가 committed까지 사용하게 되면, 
   * 신규 메모리 공간을 요구하는데 이 때 약 1초가량 jvm이 메모리 할당 중 멈춰버리는 경우가 있다. 
   * 그래서 Xms와 Xmx를 동일하게 주고 메모리를 확보한 상태에서 jvm을 기동시키곤 한다. 



# Trouble Shooting

### Memory 

 * java stack 정보 확인
 ```
 kill -QUIT <process id>
 ```


