---
layout: default
title: Java Basics
description: java의 기본적인 내용을 정리한다.
parent: Java
nav_order: 1
---

# String
new String(문자열.getBytes("charSet"), "charSet");

1. 문자열을 getBytes("charSet") 메소드
 - String을 해당 charSet으로 인코딩
 
2. String 객체를 인코딩 방식을 추가하여 생성
 - 인코딩된 문자를 Java String 객체(유니코드) 변환

정리하면, String은 유니코드형태로 저장되며, 문자열 인코딩 설정을 해 놓으면, 해당 인코딩으로 표현을 해준다.

# 시간

Java의 LocalDateTime은 JDBC type TIMESTAMP 으로 offset이 적용된 타임스탬프(타임존 데이터를 복구할 수 없는.. UTC로 저장되는 type은 TIMESTMAP_UTC 이라고 있다(또, 이것은 JDBC type이 아니고 SQL type이라고 한다. 용어가 매우 혼란스럽다.)이다. MySQL의 datetime data type은 오프셋이 적용된 타임스탬프 데이터이고, MySQL의 timestamp data type은 오프셋이 적용되지 않은 타임스탬프 데이터(UTC)로 저장된다.

Java로 작성된 프로그램이 DB를 이용할 때 사용하는 규격을 JDBC라고 한다. JDBC는 자바로 작성된 프로그램이 데이터베이스와 상호작용하기 위한 인터페이스로 각  데이터베이스 제조사가 드라이버를 제공한다.

![](/images/java/jpa-hibernate-jdbc-structure01.png)


### Resource
* [Hibernate – Mapping Date and Time](https://www.baeldung.com/hibernate-date-time)
* [Hibernate - Domain Model](https://docs.jboss.org/hibernate/orm/6.1/userguide/html_single/Hibernate_User_Guide.html#domain-model)
* [MySQL - The DATE, DATETIME, and TIMESTAMP Types](https://dev.mysql.com/doc/refman/8.0/en/datetime.html)
* [MySQL - Date and Time Literals](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html)


### 시간관련 해서 나오는 단어
* epoch
  + (중요한 사건·변화들이 일어난) 시대
  + java에서는 특정 시점(1970년 1월 1일)부터의 시간을 의미한다.
* chrono-
  + 복합형 <‘시간’, ‘시대’와 관련됨을 나타냄>
  + ChronoUnit(TemporalUnit를 구현)은 날짜와 시간을 측정하는 단위를 표현한다.
  + ChronoField(TemporalField를 구현)는 날짜 및 시간을 나타내는 데 사용되는 열거형이다. 이 열거형은 다양한 필드를 통해 **날짜와 시간의특정 부분을 나타낸다.** 여기에는 연도, 월, 일, 시간, 분 등이 포함된다. 


### 날짜와 시간의 핵심 인터페이스

![](/images/java/temporal-accessor-and-temporal-amount.png)

* **TemporalAccessor 인터페이스**
날짜와 시간을 읽기 위한 기본 인터페이스
이 인터페이스는 특정 시점의 날짜와 시간 정보를 읽을 수 있는 최소한의 기능을 제공한다.

* **Temporal 인터페이스**
`TemporalAccessor` 의 하위 인터페이스로, 날짜와 시간을 조작(추가, 빼기 등)하기 위한 기능을 제공한다. 이
를 통해 날짜와 시간을 변경하거나 조정할 수 있다.

* **TemporalAmount 인터페이스**
시간의 간격(시간의 양, 기간)을 나타내며, 날짜와 시간 객체에 적용하여 그 객체를 조정할 수 있다. 예를 들어, 특정 날
짜에 일정 기간을 더하거나 빼는 데 사용된다.

[Java's 시간처리 관련 표](https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html)



### 시간의 단위와 시간 필드
![](/images/java/temporal-unit-and-temporal-field.png)

TemporalUnit` 인터페이스는 날짜와 시간을 측정하는 단위를 나타내며, 주로 사용되는 구현체는
`java.time.temporal.ChronoUnit` 열거형으로 구현되어 있다.

TemporalField` 인터페이스는 날짜와 시간을 나타내는데 사용된다. 주로 사용되는 구현체는
`java.time.temporal.ChronoField` 열거형으로 구현되어 있다.


### 시간 관련 Trouble Shooting

##### DB 타임존(혹은 세션 타임존)과 시간이 다른 경우
어플리케이션이 동작하는 타임존 환경 설정과 DB의 타임존 설정이 다른 경우의 동작에 대해서 헷갈리기 쉽다.

TestContainer를 가지고 이러한 동작에 대해서 테스트할 수 있을거 같다.

DB가 Asia/Seoul이고, 어플리케이션 환경이 UTC인 경우


* 테스트1 - 어플리케이션에서 시간정보를 만들어서 DB에 전달하는 경우
  + 타임존 정보를 이용해서 어플리케이션의 타임존과 DB 타임존(혹은 세션)이 다르다면, 시간형태의 정보로 조회하는 경우에는 타음존을 DB 타임존(혹은 세션)에 맞춰서 변경해주기도 한다(JDBC가 이러한 동작을 하는 것을 확인)
* 테스트2 - DB에서 조회한 값을 어플리케이션 로그에서 찍어보기
  + 타임존 정보가 다를 경우, DB에서 조회한 값을 어플리케이션 조회할 때, 시간 정보가 변환된다.


### timezone setting(JDBC And Application)

##### JDBC (URL Param)

DB timezone could be set through jdbc connection string.

```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?connectionTimeZone=UTC
    username: root
    password:
```

or we can set spring boot property

```properties
spring.jpa.properties.hibernate.jdbc.time_zone=UTC
```

##### Application (JVM Default Timezone)

And we have to set JVM Default Timezone. JDBC Driver might convert the provided timestamp values from the JVM time zone to the database timezone.
This could be done through various way. 

- System Environment Variable

```
export TZ="America/Sao_Paulo"
```

- Docker Setting

```
TZ: Asia/Seoul
```

- JVM argument

```
java -Duser.timezone="Asia/Kolkata" com.company.Main
```

- Likewise, we can also set the JVM argument from the application:

```java
System.setProperty("user.timezone", "Asia/Kolkata");
```

- modify the JVM time zone from the application using the TimeZone class

```java
@PostConstruct
void started() {
  TimeZone.setDefault(TimeZone.getTimeZone("UTC"));
}
```





# Method Name

Oracle에서 정리한 메소드 이름 컨벤션을 확인할 수 있다.
* [Method Naming Conventions](https://docs.oracle.com/javase%2Ftutorial%2F/datetime/overview/naming.html)


### withXxx()
불변 객체에서 값을 변경하는 경우 `withYear()` 처럼 "with"로 시작하는 경우가 많다.
예를 들어 "coffee with sugar"라고 하면, 커피에 설탕이 추가되어 원래의 상태를 변경하여 새로운 변형을 만든다는
것을 의미한다.
이 개념을 프로그래밍에 적용하면, 불변 객체의 메서드가 "with"로 이름 지어진 경우, 그 메서드가 지정된 수정사항을
포함하는 객체의 새 인스턴스를 반환한다는 사실을 뜻한다.


### 변수의 값 초기화
* 멤버 변수: 자동 초기화
  + 인스턴스의 멤버 변수는 인스턴스를 생성할 때 자동으로 초기화된다.
  + 숫자( `int` )= `0` , `boolean` = `false` , 참조형 = `null` ( `null` 값은 참조할 대상이 없다는 뜻으로 사용
된다.)
  + 개발자가 초기값을 직접 지정할 수 있다.
* 지역 변수: 수동 초기화
  + 지역 변수는 항상 직접 초기화해야 한다.


```java
package ref;
public class InitData {
    int value1; //초기화 하지 않음
    int value2 = 10; //10으로 초기화
}
```
value1` 은 초기값을 지정하지 않았고, `value2` 는 초기값을 10으로 지정했다.

```java
public class InitMain {
    public static void main(String[] args) {
        InitData data = new InitData();
        System.out.println("value1 = " + data.value1);
        System.out.println("value2 = " + data.value2);
    }
}

<result>
value1 = 0
value2 = 10 
```
value1` 은 초기값을 지정하지 않았지만 멤버 변수는 자동으로 초기화 된다. 숫자는 `0` 으로 초기화된다.
`value2` 는 `10` 으로 초기값을 지정해두었기 때문에 객체를 생성할 때 `10` 으로 초기화된다.


### 기본형 vs 참조형
* 기본형은 들어있는 값을 그대로 계산에 사용할 수 있다.
  + 예) 더하고 빼고, 사용하고 등등, (숫자 같은 것들은 바로 계산할 수 있음)
* 참조형은 들어있는 참조값을 그대로 사용할 수 없다. 주소지만 가지고는 할 수 있는게 없다. 주소지에 가야 실체가 있다!
  + 예) 더하고 빼고 사용하고 못함! 참조값만 가지고는 계산 할 수 있는 것이 없음!
  
* 기본형을 제외한 나머지는 모두 참조형이다.
  + 기본형은 소문자로 시작한다. `int` , `long` , `double` , `boolean` 모두 소문자로 시작한다.
  + 기본형은 자바가 기본으로 제공하는 데이터 타입이다. 이러한 기본형은 개발자가 새로 정의할 수 없다. 개발
  + 자는 참조형인 클래스만 직접 정의할 수 있다.
* 클래스는 대문자로 시작한다. `Student`
  + 클래스는 모두 참조형이다.



**참고 - String**
자바에서 `String` 은 특별하다. `String` 은 사실은 클래스다. 따라서 참조형이다. 그런데 기본형처럼 문자 값을 바로
대입할 수 있다. 문자는 매우 자주 다루기 때문에 자바에서 특별하게 편의 기능을 제공한다.

### 배열 선언 방법
자바는 배열을 생성할 때, 그 내부값을 자동으로 초기화한다. 숫자는 0, boolean은 false, String은 null로 초기화된다.

배열을을 생성하면, 메모리크기만큼 메모리를 확보한다.

```java
Student[] students = new Student[]{student1, student2};

Student[] students = {student1, student2}// 더 짧게 작성 가능
```


---
# 객체 VS 인스턴스
예를 들어 student1은 객체이지만, 이 객체가 Student 클래스로부터 생성되었다는 점을 명확히 하기위해 student1을 Student의 인스턴스라고 부른다.


---
# java version
IDE같은 경우는 JDK 버전을 관리하는 방법을 제공해준다.

### 설치되어있는 자바 버전 확인및 변경하는 법
/usr/libexec/java_home -verbose

.zshrc 파일에 아래 내용 추가

export JAVA_HOME=`/usr/libexec/java_home -v 11.0.18`

---

# 생성자

생성자가 호출되었을 때, 부모 클래스의 필드가 먼저 생성되고, 자식클래스의 필드가 함께 복사된다.

먼저 super()를 작성하지 않더라도, 호출되어서 부모클래스의 생성자를 호출한다.

> 어떤 클래스가 생성될 때 그 슈퍼클래스의 생성자도 반드시 호출된다! 그래서 슈퍼클래스의 생성자가 어떤 예외를 던진다면 서브클래스의 생성자도 그 예외를 선언해야 한다.

생성자의 진짜 장점은 객체를 생성할 때 직접 정의한 생성자가 있다면 **직접 정의한 생성자를 반드시 호출**해야 한다는 점

### **기본 생성자**
매개변수가 없는 생성자를 기본 생성자라 한다.

클래스에 생성자가 하나도 없으면 자바 컴파일러는 매개변수가 없고, 작동하는 코드가 없는 기본 생성자를 자동으로 만들어준다.

**생성자가 하나라도 있으면 자바는 기본 생성자를 만들지 않는다.**

### **this()**

아래 두 생성자를 비교해 보면 코드가 중복 되는 부분이 있다.

```java
public MemberConstruct(String name, int age) {
    this.name = name;
    this.age = age;
    this.grade = 50;
}
public MemberConstruct(String name, int age, int grade) {
    this.name = name;
    this.age = age;
    this.grade = grade;
}
```

바로 다음 부분이다.

```java
this.name = name;
this.age = age;
```

이때 `this()` 라는 기능을 사용하면 생성자 내부에서 자신의 생성자를 호출할 수 있다. 참고로 `this` 는 인스턴스 자신
의 참조값을 가리킨다. 그래서 자신의 생성자를 호출한다고 생각하면 된다.

코드를 다음과 같이 수정할 수 있다.

**MemberConstruct - this() 사용**

```java
package construct;
public class MemberConstruct {
    String name;
    int age;
    int grade;
    MemberConstruct(String name, int age) {
        this(name, age, 50); //변경
    }

    MemberConstruct(String name, int age, int grade) {
        System.out.println("생성자 호출 name=" + name + ",age=" + age + ",grade=" + grade);
        this.name = name;
        this.age = age;
        this.grade = grade;
    }
}


### 오버로딩
관련 키워드: polymorphism
같은 이름의 메소드를 여러개 선한할 수 있다.

### 오버라이딩
관련 키워드: polymorphism
부모와 자식간에 동일한 a()로 코드를 작성하면, 같은 이름으로 2개가 만들어지는 게 아니라, 부모의 a()에 자식에서 작성한 메소드가 덮어씌워진다.

# 접근 제어자 

### 필드, 메서드
* private
  + 클래스 안으로 속성과 기능을 숨길 때 사용, 외부 클래스에서 해당 기능을 호출할 수 없다.
* default (package-private)
  + 나의 패키지 안으로 속성과 기능을 숨길 때 사용, 외부 패키지에서 해당 기능을 호출할 수 없다.
* protected
  + 상속 관계로 속성과 기능을 숨길 때 사용, 상속 관계가 아닌 곳에서 해당 기능을 호출할 수 없다.
* public
  + 기능을 숨기지 않고 어디서든 호출할 수 있게 공개한다.

### 클래스
* 클래스 레벨의 접근 제어자는 `public` , `default` 만 사용할 수 있다.
  + `private` , `protected` 는 사용할 수 없다.
* `public` 클래스는 반드시 파일명과 이름이 같아야 한다.
  + 하나의 자바 파일에 `public` 클래스는 하나만 등장할 수 있다.
  + 하나의 자바 파일에 `default` 접근 제어자를 사용하는 클래스는 무한정 만들 수 있다.

### instanceof

`instanceof` 키워드는 오른쪽 대상의 자식 타입을 왼쪽에서 참조하는 경우에도 `true` 를 반환한다.
쉽게 이야기해서 오른쪽에 있는 타입에 왼쪽에 있는 인스턴스의 타입이 들어갈 수 있는지 대입해보면 된다. 대입이 가능
하면 `true` , 불가능하면 `false` 가 된다.


# 내부 클래스
중첩 클래스는 총 4가지가 있고, 크게 2가지로 분류할 수 있다.**

* 정적 중첩 클래스
* 내부 클래스 종류
  + 내부 클래스(inner class): 바깥 클래스의 인스턴스의 멤버에 접근
  + 지역 클래스(local class): 내부 클래스의 특징 + 지역 변수에 접근
  + 익명 클래스(anonymous class): 지역 클래스의 특징 + 클래스의 이름이 없는 특별한 클래스

정적 중첩 클래스는 바깥 클래스의 안에 있지만 바깥 클래스와 관계 없는 전혀 다른 클래스를 말한다. 정적 중첩 클래스는 바깥 클래스의 **인스턴스 멤버에는 접근할 수 없다.** 바깥 클래스의 클래스 멤버에는 접근할 수 있다.

내부 클래스는 바깥 클래스의 내부에 있으면서 바깥 클래스를 구성하는 요소를 말한다.

**지역 클래스**는 내부 클래스의 특징을 가진다. 지역 변수에 접근할 수 있다. 접근하는 지역 변수는 `final` 이거나 사실상 `final` 이어야 한다. 주로 특정 메서드 내에서만 간단히 사용할 목적으로 사용한다.

**익명 클래스**는 지역 클래스인데, 이름이 없다. 상위 타입을 상속 또는 구현하면서 바로 생성된다. 주로 특정 상위 타입을 간단히 구현해서 일회성으로 사용할 때 유용하다. 


### 참고(private 접근 제어자)
* **private 접근 제어자**
`private` 접근 제어자는 같은 클래스 안에 있을 때만 접근할 수 있다.

중첩 클래스도 바깥 클래스와 같은 클래스 안에 있다. 따라서 중첩 클래스는 바깥 클래스의 `private` 접근 제어
자에 접근할 수 있다.

```java
nestedClass = class nested.nested.NestedOuter$Nested
```
중첩 클래스를 출력해보면 중첩 클래스의 이름은 `NestedOuter$Nested` 와 같이 바깥 클래스, `$` , 중첩 클래스의 조
합으로 만들어진다.

정적 중첩 클래스는 바깥 클래스의 정적 필드에는 접근할 수 있다. 하지만 바깥 클래스가 만든 인스턴스 필드에는 바로
접근할 수 없다. 바깥 인스턴스의 참조가 없기 때문이다.

### 내부클래스
내부 클래스는 바깥 클래스의 인스턴스에 소속된다. 따라서 바깥 클래스의 인스턴스 정보를 알아야 생성할 수 있
다.내부 클래스는 `new 바깥클래스의 인스턴스 참조.내부클래스()` 로 생성할 수 있다.

내부 클래스는 바깥 클래스의 인스턴스에 소속되어야 한다. 따라서 내부 클래스를 생성할 때, 바깥 클래스의
인스턴스 참조가 필요하다.

`outer.new Inner()` 에서 `outer` 는 바깥 클래스의 인스턴스 참조를 가진다.

`outer.new Inner()` 로 생성한 내부 클래스는 개념상 바깥 클래스의 인스턴스 내부에 생성된다.
따라서 바깥 클래스의 인스턴스를 먼저 생성해야 내부 클래스의 인스턴스를 생성할 수 있다.

![](/images/java/inner_class01.png)

실제로 내부 인스턴스가 바깥 인스턴스 안에 생성되는 것은 아니다. 하지만 개념상 인스턴스 안에 생성된다고 이
해하면 충분하다.
실제로는 내부 인스턴스는 바깥 인스턴스의 참조를 보관한다. 이 참조를 통해 바깥 인스턴스의 멤버에 접근할 수
있다.

* ** 중첩 클래스를 사용하는 이유**
  + **논리적 그룹화**: 특정 클래스가 다른 하나의 클래스 안에서만 사용되는 경우 해당 클래스 안에 포함하는 것이 논리
적으로 더 그룹화가 된다. 패키지를 열었을 때 다른 곳에서 사용될 필요가 없는 중첩 클래스가 외부에 노출되지 않
는 장점도 있다.
  + **캡슐화**: 중첩 클래스는 바깥 클래스의 `private` 멤버에 접근할 수 있다. 이렇게 해서 둘을 긴밀하게 연결하고 불
필요한 `public` 메서드를 제거할 수 있다.



---
# static
static 키워드는 아래와 같이 붙을 수 있다.

 * Static Instance Variables
 * Static Methods
 * Static Block
 * Static Classes


### Static field (변수, 상수)
클래스를 통해서 접근을 하게 되면, 객체를 만들지 않고 외부에서 접근할 수 있다.

static으로 선언된 변수는 프로그램이 실행될 때 생성 및 초기화된다. 그렇기 때문에 객체를 생성하지 않아도 이미 변수는 생성된 상태이다

만약 static keyword를 사용하여 변수를 선언하면 멤버 변수와 다른 특성을 갖는다.

다음과 같이 객체를 통해서 접근할 수 있지만, 모든 객체들이 하나의 변수를 공유하기 때문에, 클래스를 통해서도 접근할 수 있다.

```
System.out.println(Car.sNumberOfCars);

Car car1 = new Car();
System.out.println(car1.sNumberOfCars);
```
클래스를 통해서 접근을 하게 되면, 객체를 만들지 않고 외부에서 접근할 수 있습니다(객체를 통해서 접근해도됨, 동일한 결과).

static으로 선언된 변수는 프로그램이 실행될 때 생성 및 초기화됩니다. 그렇기 때문에 객체를 생성하지 않아도 이미 변수는 생성된 상태이다.



* final static 과 static
  + final static은 변경불가이지만, static만 있는 경우는 변경할 수 있다.


### Static Methods
클래스로 호출할 수도 객체를 통해서 호출할 수도 있다.

static 메소드는 객체를 생성하지 않아도 호출할 수 있다. 이것은 메소드가 객체와 분리되어 있다는 의미입니다. 그래서 메소드 내부에서 super, this와 같은 키워드를 사용할 수 없고 클래스의 멤버 변수에 접근할 수도 없다.

static 메소드는 보통 Utils, Helper 클래스를 만들어서 그룹핑하는 효과가 있다.


### Static Class

static class도 분리라는 의미에서 위와 비슷한 특성이 있다.

static 키워드를 이용하여 class를 선언하면, 상위 클래스와 분리를 해준다.

예를 들어, Car 클래스 하위에 Wheel 클래스를 선언하였는데, 이런 구조로 선언된 Wheel 클래스를 Inner class라고 부른다. Inner class의 특징 중 하나는 상위 클래스와 연결되어 있는 것이다. 그래서 Wheel 클래스는 상위 클래스인 Car 클래스의 멤버 변수에 접근이 가능하다(Static Inner Class 혹은 Static Nested Class인 경우는 상위 클래스의 멤버 변수에 접근할 수 없다).
```
public class Car {
    public int year = 2018;

    public Car() {
    }

    public class Wheel {
        public Wheel() {
            year = 10;
        }
    }
}
```
위와 같은 특성 때문에 Inner class는 상위 클래스에서만 생성할 수 있다. 상위 클래스를 통하지 않고 외부에서 직접 Inner class를 생성할 수 없다.

만약 다음과 같이 static으로 Wheel클래스를 선언하면 Car 클래스와 분리가 됩니다. 분리되었다는 것은 Wheel 클래스에서 Car 클래스의 멤버변수에 접근할 수 없다는 말인데, 이렇게 static으로 선언된 하위 클래스를 Static nested class라고 한다.
```
public class Car {
    public int year = 2018;

    public Car() {
    }

    public static class Wheel {
        public Wheel() {
            // year = 10; // compile error!
        }
    }
}
```

Static nested class는 상위 클래스가 생성되지 않아도, 외부에서 직접 객체를 생성할 수 있다. <= 대박!

예를 들어, 다음과 같이 하위 클래스를 외부에서 직접 객체를 생성할 수 있다.
```
Car.Wheel wheel = new Car.Wheel();
```
이것이 Inner class와의 가장 큰 차이점이다.

맨 외부의 클래스를 static class로 하는 것은 불가!(컴파일 오류)


* static class를 사용하는 이유?
static class로 Inner class를 생성하면 좋은 점은 grouping 이다. 어떤 클래스와 연관된 클래스들을 하위에 선언하여 관련있는 클래스들을 모아둘 수 있다.
```
public class Car {
    public Car() {
    }

    public static class Wheel {
    }
}
```
이런 부분을 Inner class로도 할 수 있지만, 위에서 설명한 것처럼 Inner class는 상위 클래스와 연결되어있어 독립된 클래스가 아니다.


그외에 테스트를 위해서 간단하게 클래스를 생성하는 경우도 사용하는 것 같다.


---
# final
https://djkeh.github.io/articles/Why-should-final-member-variables-be-conventionally-static-in-Java-kor/


### reference
* https://stackoverflow.com/questions/7026507/why-are-static-variables-considered-evil
* https://stackoverflow.com/questions/1415955/private-final-static-attribute-vs-private-final-attribute
* http://www.javaworld.com/article/2077399/core-java/private-and-final.html
* https://en.wikipedia.org/wiki/Final_(Java)
* https://en.wikipedia.org/wiki/Static_(keyword)
* http://ojava.tistory.com/50
* https://www.baeldung.com/java-static
* https://ko.wikipedia.org/wiki/정적_변수
* https://docs.spring.io/spring/docs/5.2.1.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation


### const 키워드가 없는 이유(Java에는 왜 const(상수)가 없나요?)

**세줄요약**
* Java에서 상수가 필요하다면 static final 키워드를 쓴다. (대신에 final없이 static만 쓰는 건 지양하자)
* 상황에 따라 static final을 final로 대체하여 해당 클래스 내부 혹은 해당 객체를 이용하는 곳에서만 상수처럼 쓸 수 있는 상황도 있다. 하지만 그래도 final은 상수가 아니고, 상수처럼 쓰고 있는 것이다.
* static final로 만든 상수도 다른 프로그래밍 언어의 상수const 개념과는 개념적으로도, 물리적으로도, 때로는 문법적으로도 다르다. 하지만 그래도 final에 비하자면 매우 상수답다.


---
# 형변환 (Type Conversion)

* Widening Casting (automatically) - converting a smaller type to a larger type size
  byte -> short -> char -> int -> long -> float -> double

* Narrowing Casting (manually) - converting a larger type to a smaller size type
  double -> float -> long -> int -> char -> short -> byte

### 자동형변환

```
int a = 1;
double b = a;
System.out.println(b);  // 1.0
```

자동 형변환은 좁은 데이터 타입에서 넓은 데이터 타입일 때만 가능하다.
```
double a = 1.0;
int b = a;   // type mismatch: cannot convert from double to int
System.out.println(b);
```

### 명시적 형변환

```
double a = 1.0;
int b = (int)a;
System.out.println(b); // 1
```



---
# Class/Object Type Casting

![](/images/java/Downcasting-Geeks.jpg)



이렇게 아래에서 위로 바뀐 것을 upcasting이고

그런데 반대로 하위 객체(SeaFood)에 상위 객체(Food)를 담으려고 하면 에러가 발생함.

이럴 때는 명시적 형 변환을 해줘야 한다.
```
class Food
{
    void print()
    {
       System.out.println("Food");
    }
}


class SeaFood extends Food
{
    void print()
    {
       System.out.println("SeaFood");
    }
}



class DownCast
{
   public static void main(String[] args)
   {
       Food food = new SeaFood(); //UpCasting - 상위 Food에 하위 SeaFood를 담음(자동 형 변환)
       SeaFood sea_food = (SeaFood)food; //DownCasting - 하위 SeaFood에 상위 Food를 담음(명시적 형 변환)
       sea_food.print(); //SeaFood class의 print 출력
   }
}
```

https://www.geeksforgeeks.org/class-type-casting-in-java/

https://www.baeldung.com/java-type-casting


# Java Immutability

* 클래스를 불변으로 설계하는 이유
  + 캐시 안정성
  + 멀티 쓰레드 안정성
  + 엔티티의 값 타입


 * 객체 지향 프로그래밍에 있어서 불변객체(immutable object)는 생성 후 그 상태를 바꿀 수 없는 객체를 말한다.
 * 반대 개념으로는 가변(mutable) 객체로 생성 후에도 상태를 변경할 수 있다. 객체 전체가 불변인 것도 있고, C++에서 const 데이터 멤버를 사용하는 경우와 같이 일부 속성만 불변인 것도 있다.
 * 불변 객체를 사용하면 복제나 비교를 위한 조작을 단순화 할 수 있고, 성능 개선에도 도움을 준다. 하지만 객체가 변경 가능한 데이터를 많이 가지고 있는 경우엔 불변이 오히려 부적절한 경우가 있다.
 * 이 때문에 많은 프로그래밍 언어에서는 불변이나 가변 중 하나를 선택할 수 있도록 하고 있다.

### 대표적인 불변으로 설계된 클래스들
String, Integer, LocalDate

* java가 제공하는 기본 래퍼 클래스
  + Byte, Short, Integer, Long, Float, Double, Character, Boolean




아래 코드는 불변이 아닌 클래스입니다.
```java
class MutablePerson {
   public int age;
   public int name;

   public MutablePerson(int age, int name) {
    	this.age = age;
        this.name = name;
    }
}
```
그 이유는 외부에서 age나 name을 변경할 수 있기 때문입니다.
그럼 이 클래스를 불변 클래스로 만들어보겠습니다.
```java
class ImmutablePerson {
    private final int age;
    private final int name;

    public ImmutablePerson(int age, int name) {
    	this.age = age;
        this.name = name;
    }
}
```
위와 같이 만들면 외부에서 값을 수정할 수 없습니다. 따라서 불변객체가 됩니다.
(final 변수이므로 당연히 Setter 메서드를 작성할 수 없습니다)

### Immutabable Obejct 의 장단점
 * 객체에 대한 신뢰도가 높아집니다. 객체가 한번 생성되어서 그게 변하지 않는다면 transaction 내에서 그 객체가 변하지 않기에 우리가 믿고 쓸 수 있기 때문입니다.
 * 생성자, 접근메소드에 대한 방어 복사가 필요없습니다.
 * 멀티스레드 환경에서 동기화 처리없이 객체를 공유할 수 있습니다.

하지만, 아래와 같은 단점도 있음.
 * 객체가 가지는 값마다 새로운 객체가 필요합니다. 따라서 메모리 누수와 새로운 객체를 계속 생성해야하기 때문에 성능저하를 발생시킬 수 있습니다


### 원시타입만 있는 경우

```java
public class BaseObject {

    private int value;

    public BaseObject(final int value) {
        this.value = value;
    }

    public void setValue(int newValue) {
    	this.value = newValue;
    }

    // getter는 생략 했음
}
```

위 객체는 불변객체가 아닙니다. 앞서 설명드렸지만, 외부에서 Setter를 통해 int형 필드 value의 값을 변경할 수 있기 때문입니다.

필드가 원시 타입만 있으므로 이는 final 키워드를 사용해서 불변객체로 만들 수 있습니다.

```java
public class BaseObject {

    private final int value;

    public BaseObject(final int value) {
        this.value = value;
    }

    // getter는 생략 했음
}
```
필드에 final 키워드를 사용했으므로 Setter를 구현하는 것은 당연히 불가능합니다.
따라서 이 객체의 value를 변경하려면 재할당하는 방법밖에 없습니다.

### 참조 타입이 있는 경우
참조 타입이 있는 경우의 대부분은 final을 사용하고, Setter를 작성하지 않는 것으로는 불변객체를 만들 수 없습니다.
아래의 예를 보겠습니다.
```java
public class Animal {

    private final Age age;

    public Animal(final Age age) {
        this.age = age;
    }

    public Age getAge() {
    	return age;
    }
}

class Age {

    private int value;

    public Age(final int value) {
        this.value = value;
    }

    public void setValue(final int value) {
        this.value = value;
    }

    public int getValue() {
    	return value;
    }
}
```
Animal 클래스는 final을 사용하고, Setter를 구현하지 않았지만 불변객체가 될 수 없습니다. 왜냐하면 Animal 클래스의 필드인 Age의 값을 아래처럼 변경할 수 있기 때문입니다.

```java
public static void main(String[] args) {
    Age age = new Age(1);
    Animal animal = new Animal(age);

    System.out.println(animal.getAge().getValue());
    // Output: 1

    animal.getAge().setValue(10);
    System.out.println(animal.getAge().getValue());
    // Output: 10
}
```

즉, 불변 객체의 참조 변수 또한 불변이어야 합니다.
필드에 참조 타입이 있는 경우는 여러가지 상황이 있을 수 있습니다. 대표적으로 (1)객체를 참조할 수도 있고, (2)Array나 (3)List 등을 참조할 수 있습니다.
이 세가지 상황에 대해 불변객체를 만들어보겠습니다.

##### (1) 참조 변수가 일반 객체인 경우
이 상황은 위의 예제를 고쳐보면 될 것입니다.
따라서 이는 참조 변수인 Age도 불변객체로 만들어 해결할 수 있습니다.

```java
public class Animal {

    private final Age age;

    public Animal(final Age age) {
        this.age = age;
    }

    // getter
}

class Age {

    private final int value;

    public Age(final int value) {
        this.value = value;
    }

    // getter
}
```
이 상황을 보면 결국 "참조 변수도 불변 객체이어야 한다"라는 결론이 나옵니다.

##### (2) Array일 경우
```java
public class ArrayObject {

    private final int[] array;

    public ArrayObject(final int[] array) {
        this.array = Arrays.copyOf(array,array.length);
    }


    public int[] getArray() {
        return (array == null) ? null : array.clone();
    }
}
```

배열일 경우에는 생성자에서 배열을 받아 copy해서 저장하도록 했고,
getter를 clone을 반환하도록 하면 됩니다.
배열을 그대로 참조하거나, 그대로 반환할 경우 외부에서 배열 내부값을 변경시킬 수 있기 때문에, clone을 반환하게 되면 외부에서 값을 변경시킬 수 없습니다.
만약 원시 타입 배열이 아니고, Animal[]과 같은 형태라면 해당 객체는 불변객체이어야 합니다.

```java
public static void main(String[] args) {
	int[] array = {1, 2, 3};
      	ArrayObject arrayObject = new ArrayObject(array);

        for (int num : arrayObject.getArray()) {
            System.out.print(num + " ");
        }
        // 결과: 1 2 3

        System.out.println();
        array[0] = 999; // arrayObject의 배열 값은 변하지 않는다.

        for (int num : arrayObject.getArray()) {
            System.out.print(num + " ");
        }
        // 결과: 1 2 3
}
```

##### (3) List인 경우
List인 경우에도 Array와 마찬가지로 생성시 생성자 인자를 그대로 참조하지 않고, 새로운 List를 만들어 값을 복사하도록 해야합니다. 그리고 getter를 통해 값 추가/삭제가 불가능하도록 Collection의 unmodifiableList 메서드를 사용했습니다.

여기서 Animal은 앞서 만든 불변객체입니다.

```java
import java.util.Collections;
import java.util.List;

public class ListObject {

    private final List<Animal> animals;

    public ListObject(final List<Animal> animals) {
        this.animals = new ArrayList<>(animals);
    }

    public List<Animal> getAnimals() {
        return Collections.unmodifiableList(animals);
    }
}
```
결과

```java
public static void main(String[] args) {
    List<Animal> animals = new ArrayList<>();
    animals.add(new Animal(new Age(1)));

    ListObject listObject = new ListObject(animals);

    for (Animal animal : listObject.getAnimals()) {
        System.out.print(animal.getAge().getValue());
    }
    System.out.println();
    // Output: 1

    animals.add(new Animal(new Age(2))); // List인 animals에는 추가되지만 listObject의 List에는 추가되지 않는다

    for (Animal animal : listObject.getAnimals()) {
        System.out.print(animal.getAge().getValue());
    }
    System.out.println();
    // Output: 1
}
```

---
# 객체 비교

### ==

**== 연산자의 경우 객체의 주솟값을 비교하기 때문에**, 비교하려는 객체가 동일한 객체인지를 판별한다.

디테일하게 들어가면, Primitive Type의 객체에 대해서는 값 비교가 가능하고, Reference Type에 대해서는 주소 비교를 수행한다.

이를 더 엄밀하게 서술하면 Primitive Type의 객체는 Constant Pool의 특정한 값을 참조하는 변수이기에, 결국 Constant Pool내의 동일한 주소를 비교한다.(해당 주소가 동일하기에 ==을 사용해서 비교가 가능)



### equals()
원래 equals()의 경우 Object 클래스의 메서드이고 이는 == 연산자와 동일하게 주소값을 비교를 수행하는 메서드이다.

하지만, **equals()를 오버라이딩해서 많이 사용한다.** 흔히 우리가 문자열 비교를 위해 사용하는 equals()의 경우 String 클래스에서 오버라이딩을 함으로써 문자열 간의 비교가 가능해졌다.


### hashcode()

hashCode란 메모리 주소를 통해 만든 Hash 코드 값으로 Hash를 이용하는 Collection (HashMap, HashSet, HashTable)에서 두 객체가 같은 객체인지 비교를 위해, hashCode() 메서드의 리턴 값이 같은 지를 비교 후 equals() 메서드를 통해 비교를 합니다.

그래서 equals()만 재정의가 되어 있다면, HashMap이나 HashSet에서는 내용이 동일한 두 객체를 서로 다른 객체로 인식을 하게 됩니다.

```
class User {

    private int ID;
    private String Name;

    public User(int iD, String name) {
        super();
        ID = iD;
        Name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return ID == user.ID && Objects.equals(Name, user.Name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(ID, Name);
    }
}
public class CompareObject {

    public static void main(String[] args) {
        User user1 = new User(1,"kim");
        User user2 = new User(1,"kim");

        Map<User,String> map = new HashMap<>();

        map.put(user1, "최초등록");
        map.put(user2, "업데이트");

        System.out.println("결과: " + map.size());
        // 결과: 1
    }
}
```

### Lombok - @EqualsAndHashCode

만약 Lombok이 세팅이 되어 있다면 @EqualsAndHashCode 어노테이션을 추가하여 간단하게 구현이 가능합니다.

아래 예제와 같이 @EqualsAndHashCode(exclude = "필드명")을 이용해서 ID만 같으면 같은 객체로 인식하겠다는 설정도 간단하게 구현이 가능합니다.

```
@AllArgsConstructor
@EqualsAndHashCode(exclude = "Name")
class User {

    private int ID;
    private String Name;

}
public class CompareObject {

    public static void main(String[] args) {
        User user1 = new User(1,"kim");
        User user2 = new User(1,"lee");

        Map<User,String> map = new HashMap<>();

        map.put(user1, "최초등록");
        map.put(user2, "업데이트");

        System.out.println("결과: " + map.size());
    }
}
```



---
# Collection

### Basic

 * List
   * 리스트는 일반적으로 ArrayList 를 사용
   * 데이터의 추가, 삭제가 잦다면 LinkedList 가 나음.
   * 하지만 LinkedList 는 메모리 공간을 좀 더 차지함

 * Set
   * 중복값 저장 필요 없고 (혹은 제거해야하고), 순서가 없는 데이터는 Set 을 사용
   * Set 중에는 HashSet을 가장 많이 사용
   * 입력한 순서대로 나오게 하고 싶다면 LinkedHashSet을 사용
   * 값의 정렬이 필요하다면 TreeSet을 이용
   * HashSet, TreeSet 와 LinkedHashSet 모두 쓰레드(Thread)에 안전하지 않음. 동기화 위해서는 Collections.synchronizedSet() 이용하거나 ConcurrentSkipListSet 사용

 * Map
   * 일반적으로 HashMap 사용
   * 동기화를 위해선 HashTable 사용할 수 있지만 성능이 떨어지고
   * 좀 더 성능이 좋은 ConcurrentHashMap 사용
   * 또는 Collections.synchronizedMap() 을 사용할 수도 있는데, ConcurrentHashMap 이 낫다 함.
   * HashMap 은 정렬 기능은 없음. 정렬을 하려면 TreeMap 을 사용. 동기화는 제공 안됨.
   * 동기화 위해서는 ConcurrentSkipListMap 을 사용
   * TreeMap은 동기화 제공하지 않는다. 따라서 iterator 진행과 동시에 다른 쓰레드에서 새로운 요소를 put() 하면 ConcurrentModificationException 이 발생한다.
   * 이를 해결하기 위해서 java.util.concurrent.ConcurrentSkipListMap 을 사용한다.
   * EnumMap
     * Key 를 Enum 의 원소로 제한하므로 성능이 좀 더 뛰어남(동기화는 제공 안됨)
   * WeakHashMap
     * Cache 처럼 임시 보관용 데이터는 GC 에 의해서 내용이 제거될 수 있는 WeakHashMap 이용 가능



### Multi Thread

**기존에 사용하던 동기화 컬렉션 클래스를 병렬 컬렉션으로 교체하는 것만으로도 별다른 위험 요소 없이 전체적인 성능을 상당히 끌어 올릴 수 있다.**

동기화된 컬렉션 클래스는 컬렉션의 내부 변수에 접근하는 통로를 일련화해서 스레드 안전성을 확보했다. 하지만 이렇게 만들다 보니 여러 스레드가 한꺼번에 동기화된 컬렉션을 사용하려고 하면 동시 사용성은 상당 부분 손해를 볼 수 밖에 없다. 하지만 병렬 컬렉션은 여러 스레드에서 동시에 사용할 수 있도록 설계되었다.
ConcurrentMap에는 put-if-absent, replace, condition-remove 등을 정의하고 있다.


##### 동기화되지 않은(unsynchronized) 컬렉션
 * List: ArrayList, LinkedList
 * Map: HashMap
 * Set: HashSet
 * SortedMap: TreeMap
 * SortedSet: TreeSet
 * Since JDK 1.2
 * 문제점: Thread Safe하지 않다.


##### 동기화된(synchronized) 컬렉션
 * Vector, Hashtable, Collections.synchronizedXXX()로 생성된 컬렉션들
 * Since JDK 1.2
 * 문제점: Thread Safe하나, 두개 이상의 연산을 묶어서 처리해야 할 때 외부에서 동기화 처리를 해줘야 한다. (**Iteration**, put-if-absent, replace, condition-remove 등)


 * Thread-safe 하지만, 동시 접근이 불가능해서 성능이 떨어진다.
 * 또한 두개이상 연산을 묶어서 처리해주는 경우는 외부에서 동기화 처리 해줘야 한다.

 * 예를 들면 put-if-absent = 데이터가 없는 경우에만 추가하는 연산의 경우
   * 1. 데이터 가져오기 명령으로 데이터 없는지 확인 후
   * 2. 없는것 확인되면 추가
   * 1번 작업 진행 후 2번 실행 전 다른 쓰레드가 끼어들 수 있으므로 안전하지 않다.


##### 병렬(concurrent) 컬렉션


 * Thread-safe 함은 물론이여러 Thread 가 동시에 컬렉션에 접근할 수 있다.
 * 동기화 컬렉션은 하나의 쓰레드가 접근시 다른 쓰레드는 아예 접근 불가능하지만,
 * 병렬 컬렉션은 보관하고 있는 데이터를 여러 부분으로 나눠서 락을 걸어
 * 다른 부분에 접근 중 이라면 여러 쓰레드가 동시 접근도 가능하다.

 * 또한 iteration, put-if-absent, replace, condition-remove 등의 연산을 별도 구현하고 있어
 * 추가적인 동기화도 불필요하다.

ex : java.concurrent 패키지에서 제공하는 Concurrent... 로 시작하는 컬렉션


 * List: CopyOnWriteArrayList
 * Map: ConcurrentMap, ConcurrentHashMap
 * Set: CopyOnWriteArraySet
 * SortedMap: ConcurrentSkipListMap (Since Java 6)
 * SortedSet: ConcurrentSkipListSet (Since Java 6)
 * Queue 계열:ConcurrentLinkedQueue
 * Since Java 5
 * 특이사항:
   * Concurrent(병렬/동시성)이란 단어에서 알 수 있듯이 Synchronized 컬렉션과 달리 여러 스레드가 동시에 컬렉션에 접근할 수 있다.
   * ConcurrentHashMap의 경우, lock striping 이라 부르는 세밀한 동기화 기법을 사용하기 때문에 가능하다. 구현 소스를 보면 16개의 락 객체를 배열로 두고 전체 Hash 범위를 1/16로 나누어 락을 담당한다. 최대 16개의 스레드가 경쟁없이 동시에 맵 데이터를 사용할 수 있게 한다. (p.350)
   * 반대로 단점도 있는데, clear()와 같이 전체 데이터를 독점적으로 사용해야할 경우, 단일 락을 사용할 때보다 동기화 시키기도 어렵고 자원도 많이 소모하게 된다. 또한, size(), isEmpty()같은 연산이 최신값을 반환하지 못할 수도 있다. 하지만 내부 상태를 정확하게 알려주지 못한다는 단점이 그다지 문제되는 경우는 거의 없다.


 * Queue, BlockingQueue 인터페이스는 Java 5에서 추가되었다. (Deque, BlockingDeque는 6에서 추가되었다.)
 * Synchronized 컬렉션은 객체 자체에 락을 걸어 독점하게되고, Concurrent 컬렉션은 객체 자체 독점하기가 쉽지 않은 단점이 있지만, 장점이 훨씬 더 많다. Concurrent 컬렉션은 컬렉션 전체를 독점하기 위해서는 충분히 신경을 기울여야 한다.
 * Hash를 기반으로 하는 컬렉션은 hashCode()의 해시값이 넓고 고르게 분포되지 못하면 한쪽으로 쏠린 해시 테이블을 사용하게 되는데, 최악의 경우는 단순한 Linked List와 거의 동일한 상태가 될 수 있다.



##### 병렬 컬렉션중에 Copy On Write 이란
 * 컬렉션의 데이터를 변경하는 작업시 내부적으로 복사본을 하나 더 만들어 작업하는 방식

iterate 중에 add/remove 수행하면, 비동기화 컬렉션의 경우 ConcurrentModificationException 을 발생시키고
동기화 컬렉션의 경우 락을 걸어 쓰레드가 동시접근 할 수 없도록 구현되어있다.
반면 Copy-on-write 방식은 iterate 진행중에 add 를 수행하면 동일한 데이터 복사본을 만들어 여기에 작업한다.
iterate 는 add 되기 전 데이터 원본 대상으로 진행되며, 작업이 끝나면 사라진다.
최종적으로 add 된 데이터 복사본만 남게된다.
두 명령 실행시점의 데이터를 별도로 가지고 있으니, 락을 걸필요도 없고, 동시 접근에 의한 오류도 발생하지 않는다.
하지만 복사본을 생성하므로 성능에 문제가 된다.
따라서 크기가 작고, 반복읽기가 많은 데이터에 적합하며, 추가 삭제가 많다면 성능이 떨어짐.

 * 자바에서 java.util.concurrent 패키지에서 제공된다.
   * CopyOnWriteArraySet
   * CopyOnWriteArrayList

##### 병렬컬렉션의 Set
 * CopyOnWriteArraySet
   * The CopyOnWriteArraySet is a quite simple implementation - it basically has a list of elements in an array, and when changing the list, it copies the array.
   * Iterations and other accesses which are running at this time continue with the old array, avoiding necessity of synchronization between readers and writers (though writing itself needs to be synchronized). The normally fast set operations (especially contains()) are quite slow here, as the arrays will be searched in linear time.
   * Use this only for really small sets which will be read (iterated) often and changed seldom. (Swings listener-sets would be an example, but these are not really sets, and should be only used from the EDT anyway.)

 * Collections.synchronizedSet
   * this will simply wrap a synchronized-block around each method of the original set. You should not access the original set directly. This means that no two methods of the set can be executed concurrently (one will block until the other finishes) - this is thread-safe, but you will not have concurrency if multiple threads are really using the set. If you use the iterator, you usually still need to synchronize externally to avoid ConcurrentModificationExceptions when modifying the set between iterator calls. The performance will be like the performance of the original set (but with some synchronization overhead, and blocking if used concurrently).
   * Use this if you only have low concurrency, and want to be sure all changes are immediately visible to the other threads.

 * ConcurrentSkipListSet
   * This is the concurrent SortedSet implementation, with most basic operations in O(log n). It allows concurrent adding/removing and reading/iteration, where iteration may or may not tell about changes since the iterator was created. The bulk operations are simply multiple single calls, and not atomically - other threads may observe only some of them.
   * Obviously you can use this only if you have some total order on your elements. This looks like an ideal candidate for high-concurrency situations, for not-too-large sets (because of the O(log n)).

 * ConcurrentHashMap
   * For the ConcurrentHashMap (and the Set derived from it): Here most basic options are (on average, if you have a good and fast hashCode()) in O(1) (but might degenerate to O(n)), like for HashMap/HashSet. There is a limited concurrency for writing (the table is partitioned, and write access will be synchronized on the needed partition), while read access is fully concurrent to itself and the writing threads (but might not yet see the results of the changes currently being written). The iterator may or may not see changes since it was created, and bulk operations are not atomic. Resizing is slow (as for HashMap/HashSet), thus try to avoid this by estimating the needed size on creation (and using about 1/3 more of that, as it resizes when 3/4 full).
   * Use this when you have large sets, a good (and fast) hash function and can estimate the set size and needed concurrency before creating the map.

##### 병렬컬렉션의 Map

 * Synchronized HashMap
   * Each method is synchronized using an object level lock. So the get and put methods on synchMap acquire a lock.
   * Locking the entire collection is a performance overhead. While one thread holds on to the lock, no other thread can use the collection.

 * ConcurrentHashMap was introduced in JDK 5.
   * There is no locking at the object level,The locking is at a much finer granularity. For a ConcurrentHashMap, the locks may be at a hashmap bucket level.
   * The effect of lower level locking is that you can have concurrent readers and writers which is not possible for synchronized collections. This leads to much more scalability.
   * ConcurrentHashMap does not throw a ConcurrentModificationException if one thread tries to modify it while another is iterating over it.

 * Useful link
   * https://dzone.com/articles/java-7-hashmap-vs
