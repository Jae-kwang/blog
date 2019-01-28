---
title: requestAnimationFrame() 개념 정리하기
categories: 
  - js
  - animation
tags: 
  - window
  - javascript
keywords:
  - window
  - javascript
date: 2019-01-28 21:18:25
---

부드러운 인터렉션을 제공하기 위한 첫걸음.

<!-- more -->

사용자들이 서비스를 선택할 때에는 여러 가지 기준이 있습니다. 그중에 하나는 부드러운 인터렉션 입니다. 오늘날의 기기들은 그러한 시각적인 효과를 위해 초당 60번 화면을 다시 그립니다. 그러므로 우리는 이 60개의 화면(프레임) 안에서 시각적인 효과를 표현해야 합니다.
<br/>

{% image center final-fps.gif 출처 : https://dribbble.com/shots/1945400-FPS-frames-per-second %}

사용자들은 이 프레임 중에 하나라도 놓치면 그것을 쉽게 알아챕니다. 초당 60개의 프레임을 렌더링하려면 1개의 단일 프레임은 16ms(1,000ms/60frame)안에 수행해야 합니다. 즉, 초당 60개의 프레임을 부드러운 속도로 보여주기 위해서는 약 16ms 미만으로 프레임을 유지하는 것이 좋습니다.

---

브라우저가 화면에 무언가를 그리기까지는 여러 단계가 존재합니다.
<br/>
{% image center frame-full.jpg 출처 : https://developers.google.com/web/fundamentals/performance/rendering/?hl=ko %}

애니메이션 및 기타 작업들을 수행하는 {% hl_text primary %}Javascript{% endhl_text %}, CSS 규칙을 어떤 요소에 적용할지 계산하는 프로세스인 {% hl_text primary %}Style{% endhl_text %}, 브라우저가 요소에 어떤 규칙을 적용할지 알게 되면 화면에서 얼마의 공간을 차지하고 어디에 배치되는지 계산하는 프로세스인 {% hl_text primary %}Layout(reflow){% endhl_text %}, 픽셀을 채우는 프로세스인 {% hl_text primary %}Paint(redraw){% endhl_text %}, 이전의 작업들이 개별적인 레이어에서 진행되고 이를 합치는 프로세스인 {% hl_text primary %}Composite{% endhl_text %} 로 진행됩니다.

때에 따라 다르지만 기본적으로 한번에 그림을 그리기 위해서는 위와 같은 랜더링 파이프라인을 호출하게 됩니다. 우리는 흔히 애니메이션을 수행하기 위해 setTimeout() 또는 setInterval()을 사용합니다. 하지만 이와 같은 함수들은 주어진 시간내에 동작을 할 뿐 위에서 언급한 프레임을 전혀 고려하지 않습니다.
<br/>
{% image center settimeout.jpeg 출처 : https://developers.google.com/web/fundamentals/performance/rendering/optimize-javascript-execution?hl=ko %}

그래서 종종 프레임이 누락되고 사용자게 버벅거리는 인터렉션을 제공할 수 있습니다.

## requestAnimationFrame()

화면에서 변화가 발생할 때 개발자는 브라우저에서 정확한 시간(프레임 시작 시)에 작업을 수행해야 매끄러운 움직임을 수행할 수 있습니다.

이 메소드는 실제 화면이 갱신되어 표시되는 주기에 따라 함수를 호출해주기 때문에 자바스크립트가 프레임 시작 시 실행되도록 보장합니다.

보통 1초에 60회 정도 실행이 되지만 대부분의 브라우저는 W3C 권장사항에 따라 디스플레이 주사율과 일치하도록 실행됩니다.

setTimeout(), setInterval()은 보이지 않은 곳에서도 수행되지만, requestAnimationFrame()는 현재 창에 표시 되지 않으면 애니메이션을 중지하여 배터리 수명과 성능향상에 도움이 됩니다

즉, requestAnimationFrame()을 사용하면 브라우저가 리소스 사용을 더욱 최적화하고 애니메이션을 더욱 부드럽게 만들 수 있습니다.

> 참고 :
[REQUEST ANIMATION FRAME](https://flaviocopes.com/requestanimationframe/)
[MDN - window.requestAnimationFrame()](https://developer.mozilla.org/ko/docs/Web/API/Window/requestAnimationFrame)
[udacity](https://classroom.udacity.com/courses/ud860)