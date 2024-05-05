---
layout: default
title: Annotations
parent: Java
nav_order: 10
---

[Overview of Java Built-in Annotations](https://www.baeldung.com/java-default-annotations)

[Creating a Custom Annotation in Java](https://www.baeldung.com/java-custom-annotation)

### Annotaion 에 대한 이론적인 내용

[인터페이스 강요로부터 자유를...](https://www.nextree.co.kr/p5864/)

https://velog.io/@jkijki12/annotation  <= 이 자료도 도움이 될 수 있다!



---
# Java Annotation

### 간단한 사용예

어노테이션을 정의한다.
```
// 어노테이션 생성
	import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;

    @Retention(RetentionPolicy.RUNTIME)	// 런타임중에도 유효한 어노테이션임을 기술
    public @interface Count100 {	// 어노테이션은 @interface 인터페이스명으로 정의
	
    }
```

어노테이션을 클래스에서 사용한다. (타겟에 적용)
```
// 커스텀 어노테이션을 메소드에 적용
	public class MyHello {
        @Count100
        public void hello(){
            System.out.println("hello");
        }
    }
```

어노테이션을 이용하는 코드를 수행한다.
```
// 어노테이션이 적용된  부분인지 체크하여 코드내에서 사용

    import java.lang.reflect.Method;

    public class MyHelloExam {
        public static void main(String[] args) {
            MyHello hello = new MyHello();

            try{
                Method method = hello.getClass().getDeclaredMethod("hello");
				if(method.isAnnotationPresent(Count100.class)){
                    for(int i = 0; i < 100; i++){
                        hello.hello();
                    }
                }else{
                    hello.hello();
                }
            }catch(Exception ex){
                ex.printStackTrace();
            }       
        }
    }
```

이런식으로 커스템 어노테이션을 만들어 클래스안에서 타겟에 적용하고,

호출부에서 어노테이션이 존재하는지를 확인하면서 흐름을 제어해갈수 있다.



