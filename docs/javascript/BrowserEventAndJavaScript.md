---
layout: default
title: Browser Event And JavaScript
parent: JavaScript
nav_order: 3
---


# event

브라우저의 이벤트 관리와 JavaScript의 이벤트 처리에 대해서 정리

![](/images/javascript/browser-events-01.jpg)


 * 브라우저에서의 이벤트를 JavaScript가 포착할 수 있도록 하는 위쪽 그림과 같은 이벤트 처리기들이 있습니다.
 * 이벤트가 발생할 때 실행되는 함수를 이벤트핸들러(Event Handler) 또는 이벤트리스너(Event Listener)라고 합니다.
 * 브라우저는 이벤트 리스너를 호출할 때, 사용자로부터 어떤 이벤트가 발생했는지에 대한 정보를 담은 이벤트 객체를 생성해서 리스너 함수에 전달합니다.

### 1. DOMContentLoaded 이벤트

 * DOM Tree 분석이 끝나면 DOMContentLoaded 이벤트가 발생하며, 그 외 모든 자원이 다 받아져서 브라우저에 렌더링(화면 표시)까지 다 끝난 시점에는 Load가 발생합니다.
 * 이를 이해하고, 필요한 시점에 두 개의 이벤트를 사용해서 자바스크립트 실행을 할 수 있습니다.
 * 보통 DOM tree가 다 만들어지면 DOM APIs를 통해서 DOM에 접근할 수 있기 때문에, 실제로 실무에서는 대부분의 자바스크립트코드는 DOMContentLoaded 이후에 동작하도록 구현합니다.


### 2. event bubbling

![](/images/javascript/Event_Bubbling.png)

 * 만약 img, li, ul에 각각 이벤트를 등록했었다면, 3개의 이벤트 리스너가 실행되고
 * 위 이미지로 보면, 하위엘리먼트는 3번부터 이벤트가 발생하고 2,1 순으로 이벤트가 발생함.
 * 비슷하게 Capturing이라는 것도 있습니다. 반대로 이벤트가 발생하는데,
 * 기본적으로는 Bubbling 순서로 이벤트가 발생합니다.
 * Capturing 단계에서 이벤트 발생을 시키고 싶다면 addEventListener 메서드의 3번째 인자에 값을 true로 주면 됩니다.


### 3. event delegation
|![](/images/javascript/EventDelegation-1.png)|
|:--:|
|*EventDelegation*|

|![](/images/javascript/EventDelegation-2.png)|
|:--:|
|*EventDelegation*|



# ajax (XMLHTTPRequest 통신)

 * 브라우저 새로고침 없이 서버로부터 데이터를 받는 것이 좋겠다는 생각에서 출발
 * 초기 로딩시에 모든 컨텐츠를 받는다면 시간이 오래걸림.

```javascript
 function ajax() {
   var oReq = new XMLHttpRequest();

  oReq.addEventListener("load", function() {
    console.log(this.responseText);
  });

   oReq.open("GET", "http://www.example.org/example.txt");
   oReq.send();
}
```

 * 4라인의 익명함수는 8라인, 9라인보다 더 늦게 실행되는 함수로
 * 이 익명 함수는 비동기로 실행되며, 이벤트큐에 보관되다가 load이벤트가 발생하면(서버로부터 데이터를 브라우저가 받으면)
 * 그때 call stack 에 실행되고 있는 함수가 없어서 비어있다면 stack에 올라와서 실행됨.


![](/images/javascript/ajax-flow.png)
 * 위 그림은 AJAX통신(jQuery라이브러리를 사용)을 코드단위로 어떻게 비동기로 처리되는지 보여줍니다.

```javascript
var json객체로변환된값 = JSON.parse("서버에서 받은 JSON 문자열");
```
 * 브라우저에서는 JSON객체를 제공하며. 이를 활용해서 자바스크립트 객체로 변환할 수가 있습니다.
