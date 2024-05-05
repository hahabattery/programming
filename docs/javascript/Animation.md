---
layout: default
title: Animation
parent: JavaScript
nav_order: 3
---

# Animation

 * FPS (frame per second)는 1초당 화면에 표현할 수 있는 정지화면(frame) 수입니다.
 * 매끄러운 애니메이션은 보통 60fps 입니다.
 * 따라서 16.666밀리세컨드 간격으로 frame 생성이 필요한 셈이죠. (1000ms / 60fps = 16.6666ms)

 * 간단하고 규칙적인 거 → CSS로 변경
 * 세밀한 조작이 필요한 거 → JavaScript로 변경


### 1. setInterval
```javascript
const interval = window.setInterval(()=> {
  console.log('현재시각은', new Date());
},1000/60);

window.clearInterval(interval);
```

 * 주어진 시간간격으로 함수를 실행한다.

![](/images/javascript/setInterval-problem.png)

 * 그림을 보면 제때 일어나야 할 이벤트 콜백이 지연되고, 없어지고 하는 것을 볼 수 있습니다.
   * 그림에서 mouse click callback 수행중에 시간이 지나는 상황을 말하는 듯.
 * 따라서 setInterval을 사용한다고 해서 정해진 시간에 함수가 실행된다고 보장할 수 없습니다.




### 2. setTimeout

```javascript
let count = 0;
function animate() {   
  setTimeout(() => {
    if(count >= 20) return;
    console.log('현재시각은', new Date());
    count++;
    animate();
  },500);
}
animate();
```

 * setTimeout도 엄밀하게는 어떤 이유로 인해 제때에 원하는 콜백함수가 실행되지 않을 수는 있습니다.
 * 결국 setInterval과 같은 문제가 발생할 수도 있긴합니다.  
 * 하지만 그럼에도 setTimeout은 매 순간 timeout을 조절할 수 있는 코드구현으로 컨트롤 가능한 부분이 있다는 점이 setInterval과 큰 차이



* setTimeout(setInterval)의 문제
먼저 알아야 할 점은, 대부분의 브라우저는 W3C 권장사항에 따라 디스플레이 주사율과 일치한 횟수로 콜백 함수를 호출한다. 그래서 보통 60FPS로 화면을 렌더링하고, 이는 콜백의 수가 보통 1초에 60회, 16ms(0.016초)에 하나씩 실행된다.
![](/images/javascript/fps.gif)

requestAnimationFrame()이 나오기 전에는 setTimeout()이나 setInterval()을 중첩으로 사용하여 애니메이션 작업을 했다. 따라서 함수 실행이 프레임(16ms)마다 끊기지 않고 연이어 실행되어야 유저에게 부드럽게 재생되는 애니메이션 효과를 낼 수 있다.

그러나 setTimeout()과 setInterval() 모두 시간 기반의 함수가 아니며, 앞의 콜백 함수가 종료 시에 실행된다. 게다가 만약 컴퓨터 성능이 안 좋다면, 프레임(16ms) 시작 때 함수 실행이 늦어져 화면과 싱크가 맞지 않아 애니메이션이 끊겨 보이는 현상(ex. 위 gif의 15FPS)이 일어난다.


![](/images/javascript/setTimeoutFires.avif)
반면 requestAnimationFrame()는 콜백을 실행하는 시점에 DOMHighResTimeStamp가 전달되어 시간 기반으로 작동하는 함수로 프레임(16ms) 시작 때 실행을 보장한다. 

때문에 무한 스크롤을 구현할 때 setTimeout() 기반의 throttle 대신 requestAnimationFrame()을 사용해야 한다.


### 3. requestAnimationFrame
 * setTimeout은 animation을 위한 최적화된 기능이라 보기는 어렵습니다.
 * animation주기를 16.6 미만으로 하는 경우 불필요한 frame 생성이 되는 등의 문제가 생깁니다.
 * 그 대안으로 생긴 것이 바로 requestAnimationFrame입니다.

```javascript
var count = 0;
var el=document.querySelector(".outside");
el.style.left = "0px";

function run() {
   if(count > 70) return;
   count = count + 1;
   el.style.left =  parseInt(el.style.left) + count + "px";
   requestAnimationFrame(run);
}

requestAnimationFrame(run);
```

 * 먼저 위처럼 requestAnimationFrame을 한번 실행시켜줘야 합니다.
 * 그 이후에 특정 조건이 될 때까지(함수의 탈출 조건) 계속 함수를 연속적으로 호출하는 것이죠.
 * 이렇게 requestAnimationFrame함수를 통해서 원하는 함수를 인자로 넣어주면, 브라우저는 인자로 받은 그 비동기 함수가 실행될 가장 적절한 타이밍에 실행시켜줍니다.
 * 우리는 어느 정도 브라우저를 믿고 이 함수를 전달해주는 것입니다.


### 4. CSS3 transition

 * transition 을 이용한 방법입니다.
 * 이 방법이 JavaScript로 구현하는 것보다 더 빠르다고 알려져 있다.  
 * 특히 모바일 웹에서는 transform을 사용한 element의 조작을 많이 구현합니다.


### 5. GPU를 이용한 css3 애니메이션 관련 속성들
 * transform : translateXX();
 * transform : scale();
 * transform : rotate();
 * opacity

 * 관련 링크 : https://d2.naver.com/helloworld/2061385




