---
layout: default
title: Modern Java 8
parent: Java
nav_order: 10
---

Java 8 버전에 대한 요약
Java 8 이전 버전에 대한 내용은 맨 하단에 있다.

# Lamda

> 자바에서 하나의 메소드를 가지는 인터페이스의 경우에는 기존의 익명 클래스 방식대신 람다식으로 작성할 수 있다.


 * Lambda에 대한 이해하기 쉬운 설명
   * https://jdm.kr/blog/181

 * local variables
![](/images/java/local-variable-1.png)
![](/images/java/local-variable-2.png)
![](/images/java/local-variable-3.png)


### Method Reference

 * shortcut for writing the Lamda Expressions
 * Refere a method in a class


 * Synctax of Method Reference
   * ClassName::instance-methodName
   * ClassName::static-methodName
   * Instance::methodName

![](/images/java/method-reference-1.png)
![](/images/java/method-reference-2.png)
![](/images/java/constructor-reference.png)


### Default/Static method
![](/images/java/default-method-1.png)
![](/images/java/default-method-2.png)


 * static method
   * default method와 유사하지만, This cannot be overriden by the implementation classed.

 * default method는 overriden 될 수 있지만,
 * static method는 overriden 될 수 없다.


 * class와 interface의 다른 점 (java 8)
![](/images/java/java8-classes-interfaces.png)


# Stream
 * javadoc about stream
   * https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#package.description


 * stream 처리시의 예외처리
   * https://browndwarf.tistory.com/88

### Stream API

 * map()
   * Convert(transform) one type to another.

 * flatMap()
   * Converts(Transforms) one type to another as like map()
   * Used in the context of Stream where each element in the stream represents multiple elements.
   * Example of Each Stream element represents multiple elements.
     * Stream < List >
     * Stream < Array >

 * distinct()
   * Returns a stream with unique elements

 * count()
   * Returns a long with the total number of elements in the Stream

 * sorted()
   * Sort the elements in the stream

 * filter()
   * filters the elements in the stream
   * input to the filter is a Predicate Functional Interface.

 * reduce() - terminal operation
   * Used to reduce the contents of a stream to a single value. <= 1 value!!!!
   * It takes two parameters as an input.
     * First parameters - default or initial value
	 * Second parameter - BinaryOperator<T>

 * limit(n)
   * limits the "n" numbers of elements to be processed in the stream

 * skip(n)
   * skips the "n" number of elements from the stream


 * findFirst(), findAny()
   * Used to find an element in the stream
   * Both the functions returns the result of type Optional
     * findFirst()
       * Returns first element in the stream.
     * findAny()
       * Returns the first encountered element in the stream. 꼭 첫번째라는 보장이 없음


 * Short Circuiting
![](/images/java/short-circuiting-1.png)
![](/images/java/short-circuiting-2.png)


 * of(), iterate(), generate()
![](/images/java/of-iterate-generate.png)

 * Numeric Streams - ranges(), rangeClosed()
![](/images/java/Number-Stream-1.png)

 * Numeric Streams - Boxing and Unboxing
![](/images/java/boxing-unboxing.png)

 * Numeric Streams - mapToObj(), mapToLong(), mapToDouble()
   * mapToObj()
     * convert a each element numberic stream to some Object.
   * mapToLong()
     * convert a numeric stream to a Long Stream.
   * mapToDouble()
     * convert a numeric stream to a Double Stream.

##### Terminal Operation

 * anyMatch(), allMatch(), noneMatch()
   * All these takes in a predicate as an input and returns a Boolean as an output
   * anyMatch()
     * Returns true if any one of the element matches the predicate, otherwise false.
   * allMatch()
     * Returns true if all the element in the stream matches the predicate, otherwise false
   * noneMatch()
     *Just opposite to allMatch(). Returns true if none of the element in the stream matches the predicate, otherwise false.

 * count()

 * forEach()
   * operation performs an action for each element in the stream

 * reduce()
   * Optional<T> reduce(BinaryOperator<T> accumulator)
   * T reduce(T identity, BinaryOperator<T> accumulator)
   * U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)

 * min(comparator), max(comparator)



 * collect()
   * collect() operation accumulates elements in a stream into a container such as a collection

 * Collectors.maxBy(), Collectors.minBy()
   * Comparator as an input parameter and Optional as an output

 * Collectors.summingInt()
   * this collector returns the sum as a result.

 * Collectors.averagingInt()
   * this collector returns the average as a result.

 * Collectors.groupingBy()
   * equivalent to the groupBy() operation in SQL
   * Used to group the element based on a property.
   * The output of the groupingBy() is going to be a Map<K, V>
   * 3 version
     * groupingBy(classifier)
       * groupingBy(classifier, downstream)
       * groupingBy(classifier, supplier, downstream)

 * Collectors.partitioningBy()
   * partitioningBy() collector is also a kind of groupingBy()
   * partitioningBy() accepts a predicate as an input
   * Return type of the collector is going to be a Boolean
   * 2 version
     * partitioningBy(predicate)
     * partitioningBy(predicate, downstream) // downstream -> could be of any collector




# Optional

 * Null Pointer  Exception과 Null Check를 피하기 위한 용도





# Date/Time Libraries

![](/images/java/datetime-java8-1.png)


### Calendar, Date 클래스의 문제점 (출처 : https://d2.naver.com/helloworld/645609 )

1. 불변 객체가 아니다.

-> 즉 set으로 변경이 가능하다는 점은 누군가 악의적으로 변경할 수 있기 때문에 get/set 메서드에서 직접 Date클래스를 사용하는 것이 위험하다.

2. 상수 필드 남용

-> calendar.add(Calendar.SECOND, 2);

3. 헷갈리는 월 지정

-> 1월을 0으로 표현하는 문제 + Calendar.OCTOBER로 월을 지정하지만 실질적인 값은 9(!=10)인 문제

4. 일관성 없는 요일 상수

-> 어디서는 일요일이 0, 어디서는 일요일이 1

5. Date와 Calendar 객체의 역할 분담

-> 다소 치명적인데 년/월/일 계산은 Date 클래스만으로는 부족해서 왔다갔다 하는 문제가 있다. 또한 Calendar객체를 생성하고 Date 객체를 생성하는 프로세스를 거치기 때문에 번거롭고 생성비용이 비싸다.

6. 기타 java.util.Date 하위 클래스의 문제

위와 같은 문제들이 존재한다.

그렇기 때문에 기존의 개발자들도 자바에서 제공하는 API를 사용하지 않고 적절하게 만든 오픈 소스 라이브러리인 joda time을 주로 사용한다.


### LocalDate, LocalTime, LocalDateTime ( from Java8 )

 * joda time의 영향을 받아서 비슷하게 설계됨




 * Period
Period 클래스는 두 날짜 사이의 간격을 년/월/일 단위로 나타낸다.

```Java
LocalDate startDate = LocalDate.of(1939, 9, 1);
LocalDate endDate = LocalDate.of(1945, 9, 2);

Period period = Period.between(startDate, endDate);

System.out.println("Years: " + period.getYears());
System.out.println("Months: " + period.getMonths());
System.out.println("Days: " + period.getDays());

Years: 6
Months: 0
Days: 1
```

시작 날짜와 종료 날짜 없이 of() 메서드를 통해서 바로 Period 객체를 생성가능함

```java
Period period = Period.of(6, 0, 1);

System.out.println("Years: " + period.getYears());
System.out.println("Months: " + period.getMonths());
System.out.println("Days: " + period.getDays());

Years: 6
Months: 0
Days: 1
```

Peroid은 PnYnMnD 포멧을 사용하고 있으며, parse() 정적 메소드를 사용해서 객체 생성 가능하다

```java
Period period = Period.parse("P6Y1D");
System.out.println(period);
System.out.println("Years: " + period.getYears());
System.out.println("Months: " + period.getMonths());
System.out.println("Days: " + period.getDays());

P6Y1D
Years: 6
Months: 0
Days: 1
```
![](/images/java/period-1.png)
![](/images/java/period-2.png)


 * Duration
Duration 클래스는 두 시간 사이의 간격을 초나 나노 초 단위로 나타낸다.

```java
LocalTime start = LocalTime.of(10, 35, 40);
LocalTime end = LocalTime.of(10, 36, 50, 800);

Duration duration = Duration.between(start, end);

System.out.println("Seconds: " + duration.getSeconds());
System.out.println("Nano Seconds: " + duration.getNano());

Seconds: 70
Nano Seconds: 800
```

시작 시간과 종료 시간 없이 Duration 클래스의 ofXxx() 정적 메서드를 사용하면 다음과 같이 바로 Duration 클래스를 생성할 수 있습니다. Duration 클래스를 통해 제어할 수 있는 가장 큰 시간 단위는 “일” 입니다. 그보다 큰 시간 단위는 Period 클래스를 통해서 제어가 가능하며 다음 섹션에서 자세히 다루도록 하겠습니다.

```java
Duration ofMinutes = Duration.ofMinutes(1);
System.out.printf("1 minute is %d seconds\n", ofMinutes.getSeconds());

Duration ofHours = Duration.ofHours(1);
System.out.printf("1 hour is %d seconds\n", ofHours.getSeconds());

Duration ofDays = Duration.ofDays(1);
System.out.printf("1 day is %d seconds\n", ofDays.getSeconds());

1 minute is 60 seconds
1 hour is 3600 seconds
1 day is 86400 seconds
```

Duration은 PnDTnHnMn.nS 포멧을 사용하고 있으며, parse() 정적 메소드를 사용해서 이 포멧을 따르는 문자열로 부터 객체를 생성할 수 있다.

```java
Duration duration = Duration.parse("PT10H36M50.008S");
System.out.println(duration);
System.out.printf("Seconds: %d, Nano Secodns: %d\n", duration.getSeconds(), duration.getNano());

PT10H36M50.008S
Seconds: 38210, Nano Secodns: 8000000
```
![](/images/java/duration-1.png)
![](/images/java/duration-2.png)

 * Period와 Duration의 이용방식예제

```
// get the calendar period between the times (years, months & days)
Period period = Period.between(start.toLocalDate(), end.toLocalDate());
// make sure to get the floor of the number of days
period = period.minusDays(end.toLocalTime().compareTo(start.toLocalTime()) >= 0 ? 0 : 1);

// get the remainder as a duration (hours, minutes, etc.)
Duration duration = Duration.between(start, end);
// remove days, already counted in the period
duration = duration.minusDays(duration.toDaysPart());
```

 * Instant
![](/images/java/instant-1.png)

 * ZonedDateTime
![](/images/java/ZonedDateTime-1.png)

![](/images/java/time-formatter-1.png)
![](/images/java/time-formatter-2.png)


# Java 5

### 오토 박싱(Auto-boxing), 오토 언박싱(Auto-Unboxing)
개발자들이 오랜기간 개발을 하다 보니 기본형을 래퍼 클래스로 변환하거나 또는 래퍼 클래스를 기본형으로 변환하는
일이 자주 발생했다. 그래서 많은 개발자들이 불편함을 호소했다.
자바는 이런 문제를 해결하기 위해 자바 1.5부터 오토 박싱(Auto-boxing), 오토 언박싱(Auto-Unboxing)을 지원한
다.


```java
Integer boxedValue = value; // 오토 박싱(Auto-boxing)
// Wrapper -> Primitive
int unboxedValue = boxedValue; // 오토 언박싱(Auto-Unboxing)
```

오토 박싱과 오토 언박싱은 컴파일러가 개발자 대신 `valueOf` , `xxxValue()` 등의 코드를 추가해주는 기능이다.
