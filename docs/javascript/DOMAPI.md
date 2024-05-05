---
layout: default
title: DOM API
parent: JavaScript
nav_order: 3
---

# DOM관련 javascript의 동작 특성

[defer, async 스크립트](https://ko.javascript.info/script-async-defer)

[e.preventDefault()와 e.stopPropagation()의 차이](https://pa-pico.tistory.com/20)




# DOM API

 * document 으로 사용할 수 있는 APIs
   * https://www.w3schools.com/jsref/dom_obj_document.asp
 * element 으로 사용할 수 있는 APIs
   * https://www.w3schools.com/jsref/dom_obj_all.asp


### 1. 몇 가지 유용 DOM 엘리먼트 속성

 * tagName : 엘리먼트의 태그명 반환
 * textContent : 노드의 텍스트 내용을 설정하거나 반환
 * nodeType : 노드의 노드 유형을 숫자로 반환

### 2. DOM 탐색 속성

유용한 몇명 속성들...
 * childNodes
   * 엘리먼트의 자식 노드의 노드 리스트 반환(텍스트 노드, 주석 노드 포함)
 * firstChild
   * 엘리먼트의 첫 번째 자식 노드를 반환
 * firstElementChild
   * 첫 번째 자식 엘리먼트를 반환
 * parentElement
   * 엘리먼트의 부모 엘리먼트 반환
 * nextSibling
   * 동일한 노드 트리 레벨에서 다음 노드를 반환
 * nextElementSibling
   * 동일한 노드 트리 레벨에서 다음 엘리먼트 반환

### 3. DOM 조작 메소드

 * removeChild()
   * 엘리먼트의 자식 노드 제거
 * appendChild()
   * 마지막 자식 노드로 자식 노드를 엘리먼트에 추가
 * insertBefore()
   * 기존 자식노드 앞에 새 자식 노드를 추가
 * cloneNode()
   * 노드를 복제
 * replaceChild()
   * 엘리먼트의 자식 노드 바꿈
 * closest()
   * 상위로 올라가면서 가장 가까운 엘리먼트를 반환


### 4. HTML을 문자열로 처리해주는 DOM 속성 / 메소드

 * innerText
   * 지정된 노드와 모든 자손의 텍스트 내용을 설정하거나 반환
 * innerHTML
   * 지정된 엘리먼트의 내부 html을 설정하거나 반환
 * insertAdjacentHTML()
   * HTML로 텍스트를 지정된 위치에 삽입

