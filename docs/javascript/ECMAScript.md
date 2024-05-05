---
layout: default
title: ECMAScript
parent: JavaScript
nav_order: 3
---


ES = ECMAScript

 * ECMA 는 정보통신시스템에 대한 표준을 제정하는 기구
 * javascript는 다양한 브라우저에서 지원하기 위한 스크립트 표준
 * ECMAScript는 ECMA 인터내셔널에 의해 제정된 ECMA-262 기술 규격에 의해 정의된 범용 스크립트 언어
 * JavaScript = ECMAScript이라는 1개의 코어 + BOM(Browser Object Model) + DOM(Document Object Model)


* ES에 대한 이해를 위한 검색 키워드
  + ES3 vs ES5 vs ES6
  + scope chain vs Lexical Environment

---
# ES3 (1999)

 * 대중적으로 언급할 때의 자바스크립트가 ES3.

## ES3의 특징
 * 함수 단위의 스코프
 * 호이스팅
 * 클로저
 * 프로토타입
 * 대부분의 브라우저에서 지원

---
# ES5 (2009)
 * 배열
   + 배열과 관련하여 편리한 메소드들이 다수 생겼다. forEach, map, reduce, filter, some, every와 같은 순환 메소드들이 생겼다. 이 메소드들은 개발 시 불필요한 중복 코드를 줄여주어서 가독성은 높이고 버그율은 낮추는 효과가 있다.
 * 객체
   + 객체는 프로퍼티에 대한 설정을 할 수 있게 되었다. 객체를 생성, 수정, 복사하는 표준 메소드 Object.Create(), Object.defineProperty(), Object.freeze(), Object.assign() 등 과 getter, setter 등이 추가되었으며, Object.keys() 메소드를 이용하면 for in 메소드도 대체할 수 있게 되었다.
 * strict 모드
   + 문법을 좀 더 깐깐하게 체크하는 모드이다. 너무 자유분방하였던 기존 ES를 안전하고, 개발자가 인지할 수 있는 범위 안에서 개발할 수 있도록 사용하기 위해 등장했다. Strict mode - JavaScript | MDN에서 자세한 특징을 확인 할 수 있다.
 * bind()
   + 메소드 this를 강제로 바인딩 시켜주는 메소드이다. 좀 더 명확하게 this 스코프를 지정 할 수 있게 되었다.



---
# ES6 (2015)

 * TypeScript의 기반이 되는, 클래스 문법과 모듈 기능 추가, IE9 부터 지원

### ES6의 특징
 * 호이스팅이 사라진 것 같은 효과
 * 함수 단위 스코프에서 블록 단위 스코프로 변경
 * this를 동적으로 바인딩하지 않는 화살표 함수
 * 모듈화 지원
 * 콜백 지옥에서 구원해줄 Promise
 * Default, Rest 파라미터
 * 해체 할당, Spread 연산자
 * 템플릿 리터럴
 * 브라우저(특히 MS 계열)에서 지원해주지 않는 경우가 많아 바벨(Babel)이라는 트랜스파일러를 써야한다.

### var let const
 * https://gist.github.com/LeoHeo/7c2a2a6dbcf80becaaa1e61e90091e5d
 * var는 function-scoped
 * let, const는 block-scoped
   * es2015에서는 let, const가 추가
   * let과 const의 차이점은 변수의 immutable여부이다.

 * let은 변수에 재할당이 가능하지만,
 * const는 변수 재선언, 재할당 모두 불가능하다

### Static Method vs Instance Method
 * OOP에서 static method는 객체생성없이 호출할 수 있고, instance method는 객체생성후 호출할 수 있다.
 * 문법이 정형화되어있지 않은 JavaScript의 특성상, 이러한 클래스나 객체관련한 동작은 simulate하는 것이다
   * https://abdulapopoola.com/2013/03/30/static-and-instance-methods-in-javascript/


---
# ES7 (2016)

 * 제곱 연산자(**) 등장
 * Array.includes 배열에 해당 요소가 존재하는지 확인하는 메소드 등장


---
# ES8 (2017)
 * ES2017에서는 Promise 급의 중대한 변화인 async, await등이 발표됨
   + async
   + await
 * 객체의 좀더 심화된 메소드가 등장
   + -Object.keys()에 대응되는 메소드인  Object.values()
   + -Object.keys()와 Object.values()를 합쳐 놓은 Object.entries() 등
 * 문자열 단순 편의기능이 추가:
   + 문자열 앞부분에 공백을 넣어 자리수를 맞춰주는 String.padStart()
   + 문자열 뒷부분에 공백을 넣어 자리수를 맞춰주는 String.padEnd()
   + 매개변수 마지막에 콤마를 붙이는 것 허용

# Javascript 엔진 관점의 분석

* ES3와 ES5의 가장 큰 차이점
ES3에서는 Scope Chain, ES5에서는 Lexical Environments개념이 가장 큰 차이점

### 실행 컨텍스트(Execution Context)와 식별자 해결(Identifier Resolution)
엔진 처리는 크게 해석과 실행으로 나눌 수 있다.

해석이란 컴파일과 실행할 환경을 설정하는 것이고(Context),  실행이란 해석단계에서 설정된 환경을 바탕으로 코드를 실행하는 것입니다.(Execution)  그렇기에 실행 환경을 만드는 것은 함수가 호출되기 전/후로 가능하지만, 실행은 함수가 호출 된 다음에 가능합니다.(Identifier Resolution) 즉, 함수라는 단위를 어떻게 묶어서 취급할 것이냐라는 것이다.  

예를 들어, 여러박스에 함수에 있는 내용을 분산시키면 실행될수도 없고 올바르지 않다.  반면, 하나의 박스(Context)안에 함수에서 발생할 수 있는 것을 모두 넣을 수 있으면 메모리에도 하나의 박스만 올리면되기에 간단해진다.
함수가 호출되었을 때 어떻게 실행하는 모음을 가져갈 것인지에  대한것이 Execution Context이다.  그런데 어째서 Execution Context를 묶음으로 가져가야할까요?  그이유는 식별자 해결(Identifier Resolution)에 있다.
함수가 호출되었을 때 함수의 이름을 찾아야하는데 그게 Execution Context안에 있다면 찾기가 쉬워진다.
하지만, Execution Context밖에 존재하게 된다면 엔진처리도 복잡해지고 비용도 높아지게 된다. 그렇기에 Execution Context를 묶음으로 가져가는 것이다.

그리고 이 식별자 해결(Identifier Resolution)에서 파생된 키워드가 Scope이다. 식별자가 어디에 있는지에 대해서 구조적으로 가져가는데 이것도 엔진 관점에서 볼 때 비용이 부담되기 때문이다.

엔진의 기본적인 방향은 분산되지 않고 컨텍스트 개념으로 하나의 묶음 안에서 동작하겠다는 것이다.

결국, 식별자 해결을 하기위해 컨텍스트가 필요하고 함수가 호출되었을 때 식별자 해결을 하게되면서 실행 컨텍스트로 묶어버리는 것입니다. 그리고 이걸 어떤 단위로 묶을 것인가에 대해서 ES3에서는 Scope Chain 개념을 사용했고, ES5에서는 Lexical Environments라는 개념을 사용했다.
 Scope Chain 과 Lexical Environments 개념은 컨텍스트가 다릅니다. 컨텍스트는 하나의 묶음으로 가져가는게 최선인데, 여러개로 분산되어있고 각각의 요소(Scope)마다 찾아가서 자원을 가져오는 것은 비용소모가 크다.
