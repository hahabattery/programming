---
layout: default
title: AOP AspectJ
grand_parent: Springs
parent: Spring Frameworks
nav_order: 4
---

AspectJ는 Spring과 별도의 라이브러리임.

Spring AOP API에서 제공하지 않은 필드에 대한 Advisor를 지원하고, CTW(Compile Time Weaving), LTW(Load Time Weaving)과 같은 다양한 위빙 방법을 제공해 프로그램의 퍼포먼스를 향상 시킬 수 있도록 해준다.

또한 @Aspect 어노테이션을 바탕으로 로직을 작성할 수 있어, xml 방식보다는 더 편리하다.

# 리소스
* https://dahye-jeong.gitbook.io/spring/spring/2020-04-10-aop-aspectj  <= Aspectj에 대해서 잘 정리되 있다

# 동작 방식
AspectJ는 두번의 컴파일 과정이 필요합니다.
1차적으로 Java 코드를 컴파일해서 binary 형태의 class 파일을 만들고
2차적으로 binary 파일들을 분석해서 필요한 부분에다가 aspect를 삽입하는 weaving 과정을 거칩니다.

CTW(Compile-Time Weaving)으로 처리된다고 표현합니다.


 Proxy 모드가 갖는 제약사항을 처리할 수 있고
LTW의 경우는 class를 load하는 과정에서만 weaving을 하기 때문에 RTW보다는 성능이 빠릅니다.
CTW는 컴파일시에만 weaving을 하기 때문에 성능이 가장 좋습니다.
대신 Proxy 모드에서 추가적인 설정작업이 필요합니다.

Proxy 모드만으로도 개발하는데 큰 문제는 없습니다.
만약에 AspectJ 모드가 필요하다면 CTW, LTW 중에 한가지 방식을 골라서 설정하시면 됩니다.

이 설정도 Spring이 추구하는 방식대로 소스코드는 수정하지 않습니다.
build.gradle과 configuration 수정만으로 진행됩니다.
