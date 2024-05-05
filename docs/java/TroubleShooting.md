---
layout: default
title: Trouble Shooting
parent: Java
nav_order: 20
---

예외적인 케이스에 대한 정리


---
# String

### ValueOf() vs toString()
 * String.valueOf() - 파라미터가 null이면 문자열 "null"을 만들어서 반환한다.
 * toString() - 대상 값이 null이면 NPE를 발생시키고 Object에 담긴 값이 String이 아니여도 출력한다.



