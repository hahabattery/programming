---
layout: default
title: Advanced Topics
parent: JavaScript
nav_order: 1
---



* be hoisted
be hoisted = 일본어의 mochiagerareta 와 유사한 의미로 보인다.


# Closure

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#function_scope

> Variables defined inside a function cannot be accessed from anywhere outside the function, because the variable is defined only in the scope of the function. However, a function can access all variables and functions defined inside the scope in which it is defined.
> In other words, a function defined in the global scope can access all variables defined in the global scope. A function defined inside another function can also access all variables defined in its parent function, and any other variables to which the parent function has access.


 * inner function은 외부함수가 return되더라도 항상 외부 함수의 변수나 파라미터에 접근가능하다.

|![closure](/images/javascript/closure-1.png)|
|:--:|
|*closure - 1*|

|![closure](/images/javascript/closure-2.png)|
|:--:|
|*closure - 2*|

|![closure](/images/javascript/closure-3.png)|
|:--:|
|*closure - 3*|

|![closure](/images/javascript/closure-4.png)|
|:--:|
|*closure - 4*|
