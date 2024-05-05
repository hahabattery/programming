---
layout: default
title: Spring Data
grand_parent: Springs
parent: Spring JPA
nav_order: 4
---

# Paging

### JPA Paging 이 필요할 때 사용하려면

```java
public List<Board> findByBnoGreaterThanOrderByBnoDesc(Long bno,Pageable paging)
```

코드는 기존과 동일하지만, 파라미터에 Pageable이 적용되었고 Collection<> 에서 List<> 로 변경됨.

* Pageable 인터페이스가 적용되는 경우는 리턴타입이 Slice, Page, List 타입이다.


# TroubleShooting

### QuerySyntaxException: Unable to locate class 
* Reason
  + JPQL cann't return Static Inner Class.

```
@Query("select new com.example.service.dto.PlainOuterClass.StaticInnerClass"
      + "(p.id, p.type, p.imageUrl)"
      + " from Product p where p.id = :id")
   List<PlainOuterClass.StaticInnerClass> findByProductsId(@Param("id") Long id);
```

But recently there is some post this error use following syntax.

```
package.PlainOuterClass$StaticInnerClass
```

### No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer

fetchMode가 lazy로 되어있는 경우, 아직 조회하지 않은 상태에서 jackson library가 동작하다가 나오는 에러인 듯하다.

