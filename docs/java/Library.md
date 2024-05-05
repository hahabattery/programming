---
layout: default
title: Libraries
parent: Java
nav_order: 10
---

여러가지 유용한 라이브러리를 정리하는 페이지


pom.xml 파일 작성은 https://mvnrepository.com/ 에서 검색해서 넣으면 된다.

groupId 별로도 그룹화해서 표시해준다.






# Converting


### Jackson

https://github.com/FasterXML/jackson





# 코드 생성 (code generator)

### Lombok
https://projectlombok.org/


### MapStruct
https://mapstruct.org/documentation/stable/reference/html/


MapStruct는 Entity와 Dto간의 매핑을 지원하는 라이브러리다.

Entity와 Dto간의 매핑을 위해 getter/setter를 남발하며 직접 구현하는 것을 지원하는 라이브러리는 크게 ModelMapper와 MpaStruct가 있다.

```
@Mapper(componentModel = "spring")
public interface CodingRoomMapper extends GenericMapper<CodingRoomDto, CodingRoom> {


}
```
@Mapper : MapStruct Code Generator가 해당 인터페이스의 구현체를 생성해준다.

componentModel = "spring" : spring에 맞게 bean으로 등록해준다

build한다면 @Mapper가 붙은 interface에 대한 구현체가 자동 생성되는 것을 볼 수 있다.

```
@Service
@RequiredArgsConstructor
public class CodingRoomService {

    private final CodingRoomMapper codingRoomMapper;
    ...
        
    CodingRoomDto codingRoomDto =  codingRoomMapper.toDto(codingRoom);
    ...        
    }
```



