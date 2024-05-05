---
layout: default
title: Comparing to Python
parent: JavaScript
nav_order: 3
--- 
 
 * key value형태의 데이터타입을 보통 dictionary에 포함된다.


 * Python에서는 dictionary mapping 형태로 사용할 수 있다.

 * javascript에서는 dictionary는 object와 동일하다.

 * javascript에서는 key가 '나 "으로 key를 싸지 않아도 된다.
```
var john = {
    name: 'John',
    yearOfBirth: 1990,
    job: 'teacher'
};
```

### dictionary type에 대한 차이점 설명

```
 b = {i:j} 
```

##### Python

```
 i='k'
 j=1
 b = {i:j}
 b['k'] # -> 1 
```

 * both i and j are evaluated, but in Javascript, only j is evaluated.
 * python에서는 dictionary type 접근시에 dot notation을 사용할 수 없다. JavaScript는 사용 가능하다.
 * python에서 dictionary type의 key는 항상 '나 "를 사용해서 접근해야 한다. JavaScript는 '나 "를 사용하지 않아도 된다.

##### JavaScript

```
 i='k'
 j=1
 b = {i:j}
 b['i'] // -> 1
 b.i // -> 1
 // b[i], b['k'] and b.k are not defined 
```

 * In Javascript, using the identifier in dot notation is completely identical in all cases to using a string that "looks like" the identifier in array notation. 
 * Hence, b = { 'i':1 } ; b['i'] // -> 1    b.i // -> 1 When a number or boolean is used in a dictionary, 
 * Javascript will access the element using a string representation of the number or boolean. 
 * Not so in Python — a string and a number (or boolean) are different hashables.

```
//b[i] will be valid in this case of javascrip code
b = {}
var i = 'k';
b[i] = 2
console.log(b); // -> {'k':2}
//thus b[i] = b['k'] = 2
```


