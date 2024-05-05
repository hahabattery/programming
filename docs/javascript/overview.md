---
layout: default
title: Overview
parent: JavaScript
nav_order: 1
---


# Resources
* [Javascript Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
  + This documents is quite confused, because it doesn't differentiate old version of java (such as ES3)  and ECMAScript 3/5.
* [Microsoft JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
* [Catsb's Dlog JavaScript(JavaScript의 내부 동작에 대한 설명글)](https://catsbi.oopy.io/0f510a03-8f5e-4683-82ad-96a123b228f2) 


# inflearn의 자바스크립트 강좌
튜토리얼 완독 + inflearn 파일 다시 보고 tutorial 위주로 편집하기.

inflearn 의 자바스크립트 강좌를 정리한 내용 (https://github.com/bitedd/Examples/tree/master/JavaScript/inflearn-javascript-bible) 을 확인하던 중.
이론적인 내용이 많아서 진도를 많이 빼긴 어렵다.
“es5-advanced의 DER관련 내용을 보고 있음”  ← 내용이 익숙하지 않아서, 반복적으로 내용을 확인해야 된다.


# JavaScript의 기본적인 특징

 * variable have no type
 * functions are objects. 함수를 인자로 줄수 있다는 뜻으로, functional programming이 가능하다는 의미이다.
 * ===는 type과 value가 모두 같아야 한다.
 * java script is single thread(only one line of code can be executed at any given time).
   * Browsers(ajax) and Node.js manage other threads that do things for you.

### function

 * 함수는 다른 일반 변수들처럼 사용될 수 있기 때문에  first class citizen 이라고 한다.
 * 다른 함수를 인자로 받는 함수를 high order function 이라고 한다.



### JS is a compiled language
![](/images/javascript/js-is-a-compiled-language.svg)

 * 1. After a program leaves a developer's editor, it gets transpiled by Babel, then packed by Webpack (and perhaps half a dozen other build processes), then it gets delivered in that very different form to a JS engine.
 * 2. The JS engine parses the code to an AST(abstract syntax tree, 추상구문트리).
 * 3. Then the engine converts that AST to a kind-of byte code, a binary intermediate representation (IR), which is then refined/converted even further by the optimizing JIT compiler.
 * 4. Finally, the JS VM executes the program.


### strict mode

 * strict mode for file

```javascript
// only whitespace and comments are allowed
// before the use-strict pragma
"use strict";
// the rest of the file runs in strict mode
```

 * strict mode for function

```javascript
function someOperations() {
    // whitespace and comments are fine here
    "use strict";

    // all this code will run in strict mode
}
```

 * JS는 거의 모든 프러덕션 환경에서 항상 transpiled 된다. transpiled되는 경우에는 strict mode로 처리되게 된다.
 * ES6 modules assume strict mode, so all code in such files is automatically defaulted to strict mode.





# 테스트

### 개발환경용 웹서버 띄우기
 * http-server 모듈을 전역
   * npm install –g http-server
 * 실행할 파일 위치로 이동후
   *  http-server ./




# JavaScript 표준

 * ES5와 ES2015(ES6)는 호환성이 높으나
 * ES2016 ~ ES2018 은 호환성이 낮다


### JavaScript가 동작하는 시점

일반적으로 javascript 처리는
> When the browser encounters a classic script tag when parsing the HTML, it stops parsing and hands over to the JavaScript interpreter,
> which runs the script. The parser doesn't continue until the script execution is complete (because the script might do document.
> write calls to output markup that the parser should handle).



# 성능 테스트

```
function work() {
  const start = Date.now();
  for (let i = 0; i < 1000000000; i++) {}
  const end = Date.now();
  console.log(end - start + 'ms');
}

work();
console.log('다음 작업');
```

---
# JavaScript 디버깅하기

 * 크롬에서 개발자 모드에서 Source 탭으로 들어가면 소스가 표시되고, breakpoint를 추가할 수 있다.
 * JavaScript 소스에 debugger; 추가하면, 따로 breakpoint를 추가하지 않아도 된다.


---
# Ajax


### plain javascript



### JQuery

##### 리퀘스트바디 이용

```
$(".login-form button[type=submit]").click(login);

function login(e) {
  //원래 동작해야 할 이벤트를 중지 시킨다.  
	e.preventDefault();

  //유저가 입력한 정보를 js 오브젝트로 만든다.  
	var data = {
		'email' : $('#email').val(),
		'password' : $('#password').val()
	};

  //위에서 만든 오브젝트를 json 타입으로 바꾼다.
	var json = JSON.stringify(data);
	var url = $(".login-form").attr("action");

	$.ajax({
		type : 'post',
		url : url,
		data : json,
		contentType : "application/json; charset=utf-8",
		error : onError,
		success : onSuccess
	});
}
```

* contentType은 보내는 데이터의 타입이다.
  + application/json; charset-utf-8이 흔히 쓰인다.
  + 디폴트는 application/x-www-form-urlencoded; charset=utf-8 이다.

* dataType은 서버에서 어떤 타입을 받을 것인지를 의미한다.
  + json, html, text 등등...
  + jQuery가 이것을 이용해 success나 done 함수의 파라미터로 받아 처리



##### 쿼리 파라미터로 넘기는 경우

```
function login(e) {
	e.preventDefault();

	var data = $(".login-form").serialize();
	var url = $(".login-form").attr("action");

	$.ajax({
		type : 'post',
		url : url,
		data : data,
		error : onError,
		success : onSuccess
	});
}
```

### AJAX 함수를 Promise로 변환

 * 아래와 같은 ajax함수가 있을 때

```
function doTheThing() {
  $.ajax({
    url: window.location.href,
    type: 'POST',
    data: {
      key: 'value',
    },
    success: function (data) {
      console.log(data)
    },
    error: function (error) {
      console.log(error)
    },
  })
}
```

 * 만약에 아래처럼 동기적으로 Ajax 호출을 하려고 한다면...

```
doTheThing()
doSomethingElse()
```

 * 최소한의 노력으로 Promise로 변경할 수 있다.

```
function doTheThing() {
  return new Promise((resolve, reject) => {
    $.ajax({
      url: window.location.href,
      type: 'POST',
      data: {
        key: 'value',
      },
      success: function (data) {
        resolve(data)
      },
      error: function (error) {
        reject(error)
      },
    })
  })
}
```

 * promise chaining으로 동기적으로 호출할 수 있게 된다.
 
 ```
 doTheThing()
  .then((data) => {
    console.log(data)
    doSomethingElse()
  })
  .catch((error) => {
    console.log(error)
  })
 ```


---
# Templating

![](/images/javascript/html_templating-01.png)

 * 간단한 html templating

```javascript
var data = {  title : "hello",
              content : "lorem dkfief",
              price : 2000
           };
var html = "<li><h4>{title}</h4><p>{content}</p><div>{price}</div></li>";

html.replace("{title}", data.title)
    .replace("{content}", data.content)
    .replace("{price}", data.price)
```

 * HTML Template의 보관
   * 서버에서 file로 보관하고 Ajax로 요청해서 받아옵니다.
     * script tag의 속성에 type="text/template" 추가 필요
   * HTML코드 안에 위치시킨다.





---
# TroubleShooting

### JavaScript를 이용해서 idle에서 wake하는 처리를 어떻게 할 것인가.
보통은 이렇게 처리해버린다.
https://stackoverflow.com/questions/13798516/javascript-event-for-mobile-browser-re-launch-or-device-wake

최근에 이런 API가 새로 나왔다고 한다.
https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API


하지만, 테스트를 해보니 결국 제대로 처리가 안됨.

세션이 서버 설정에 의해서 타임아웃시간동안 활동이 없으면, 차단되는 것을 우회하려고 한 것인데.

다음에 이런 이슈가 있을 때는, 자동로그인 기능(쿠키에 토큰을 저장하는 방식; Remember Me)을 이용해서 처리하는게 나을거 같다.
