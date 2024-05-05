---
layout: default
title: Overview
parent: Low Code
nav_order: 1
---

# provide context
import 라이브러리, 이해하기 쉬운 함수/파라미터 명, 인라인 코멘트, 만약에 라이브러리의버전에 따라서 동작이 달라진다면, 문서의 내용을 코멘트해주면 된다.

# be predictable
이 말은 웹 어플리케이션을 작성시에는 Mvc 와같은 형태를 따르는게 낫다는 의미이다. Copilot이 학습할 때 많이 사용된 코드형태를 따르는 것이 copilot이 많이 도와주게 될것이다.

# 다른 부분에 영향을 많이주는 부분을 먼저 작성
* MVC 모델에서는 보통 model이 view와 controller에 영향을 두기 때문에 **model 작성을 먼저**하는게 나을 것이다.
* api개발시에는 api schema를 작성하는데 나을 것이다.


# 정확하진 않다.
인공지능은 확률적으로 옳다.

대략적인 방향은 맞을 수 있지만, 모든 세세한 부분을 모두 고려하지 못하기 때문에, 생성형AI가 학습된 시점의 데이터(프로그래밍 라이브러리버전과 같은) 버전같은 부분이 달라서 정확하지 않을 수 밖에 없다. 


# Github Copilot
* 추천 내용 수락
  + Tab
* Cycle Through Alternatives
  + Alt + [ or Alt + ]
* 첫 단어만
  + Command + ->
* 첫 라인만
  + Shift + Command + ->


# Code Whisperer
AWS에서 제공하는 모범사례들을 학습하였기 때문에, AWS 환경에서 개발할 때는 가장 높은 품질을 제공한다고 한다.
단축키는 이용하는 환경에 따라서 다를 수 있음.

* Manually trigger CodeWhisperer
  + MacOS: Option + C
  + Windows: Alt + C

* Accept a recommendation
  + Tab

* Next recommendation
  + Down or Right arrow
* Previous recommendation
  + Up or Left arrow

* Reject a recommendation
  + ESC


