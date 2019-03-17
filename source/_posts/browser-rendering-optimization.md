---
title: 브라우저 렌더링 최적화(1)
categories:
  - Browser
tags:
  - Browser
  - Rendering
  - Optimization
keywords:
  - Browser
  - Rendering
  - Optimizationy
thumbnailImage: https://raw.githubusercontent.com/Jae-kwang/blog/master/source/img/udacity.png
date: 2019-03-17 16:59:39
---

Udacity의 Browser Rendering Optimization 강의를 수강하고 나서 정리해보았습니다.
1, 2, 3 강의 내용입니다.

<!-- more -->

이 과정은 브라우저의 렌더링 파이프 라인을 파악하여 고성능 웹 응용 프로그램을 쉽게 만들 수 있도록 도와줍니다.

### 1. The Critical Rendering Path

오늘날 디바이스들은 **초당 60번** 화면을 다시 그립니다.

서비스 사용자들은 이 프레임 중 하나가 늦어진다는 것을 쉽게 알아챕니다. (화면이 버벅대는 현상)

이런 결과는 서비스의 평가를 낮출 뿐만 아니라 실제로 서비스가 멈출 수 있는 가능성까지 있습니다.

1000ms당 60프레임을 렌더링하려면 단일 프레임을 렌더링하기 위해선 약 16ms(1000/60)가 필요합니다. 
즉, 유려하고 버벅대지 않는 자연스러운 화면을 만들기 위해서는 16ms 미만으로 프레임이 구성되도록 작업이 되어야 합니다.

우선 프레임이 구축되는 순서를 살펴보겠습니다.
처음 브라우저는 서버에 GET 요청을 하고 서버는 응답으로 HTML을 보냅니다.

이때, 브라우저는 해당 HTML을 파싱하는데 이 단계를 **Chrome Dev Tools** 에서는 {% hl_text blue %}Parse HTML{% endhl_text %}로 나타내며, CSS 파싱은 {% hl_text blue %}Parse Stylesheet{% endhl_text %}, DOM과 CSS 결합은 {% hl_text purple %}Recalculate Styles{% endhl_text %}로 표현하고 있습니다.
</br>
</br>
이렇게 DOM과 CSS가 결합하면 Render Tree를 구축합니다.
Render Tree는 Dom Tree와 비슷해 보이지만 CSS를 통해 보이지 않는 요소는 제거됩니다.
즉, 페이지에 실제로 보이는 요소만 Render Tree에 표시됩니다.
</br>
</br>
브라우저가 요소에 적용되는 규칙(공간을 얼마나 차지하고, 어디에 있는지 등)을 알게 되면, 레이아웃 계산을 시작할 수 있습니다.
이는 {% hl_text purple %}Layout{% endhl_text %}으로 표현하고 있습니다. 다음 해당 픽셀에 색을 채우는 Rasterizer 단계인데 이는 {% hl_text green %}Paint{% endhl_text %}로 표현됩니다.
</br>
</br>

브라우저는 레이어라는 불리는 여러 표면을 만들고 그것들을 개별적으로 그릴 수 있습니다. 이러한 개별적인 레이어를 합치는 단계는 {% hl_text green %}Composite{% endhl_text %}로 표현합니다.

이렇게 브라우저는 렌더링 파이프라인을 가지고 있습니다.
Javascript로 위의 파이프라인을 제어할 수 있는 경우는 3가지의 경우있습니다.

{% image rendering-pipline.jpg %}

첫째, Javascipt로 요소에 변형을 가했을 경우 Style, Layout, Paint, Composite단계가 모두 발생합니다.
둘째, 색상만 변경했을 경우에는 Layout이 발생하지 않고 Style, Paint, Composite만 발생합니다.
셋째, 위치 및 색상 모두 변경되지 않는 것을 수정 했을 경우에는 Style, Composite만 발생합니다. (transform, opacity는 요소가 레이어를가지고 있을경우 Composite에서 만 처리됩니다.)

> https://developers.google.com/web/fundamentals/performance/rendering/

### 2. App Lifecycles

웹 앱 라이프 사이클은 Load, Idle, Animation, Response로 4가지의 영역이 존재합니다.

사용자는 언제나 페이지가 빠르게 Load 되기를 원합니다. 이때에는 처음 페이지가 보이기 위해 꼭 필요한 것들만 필요로 합니다. 모든 방법을 써서 1초 안에 이 과정을 완료되도록 최적화하는 것은 매우 중요합니다. (파일 크기 축소, CDN 활용 등)

앱이 완전히 Load 된 후는 보통 Idle 상태입니다. 이것은 사용자가 상호작용하기를 기다린다는 의미로, 이때는 Load 시간을 1초로 맞추기 위해 뒤로 미뤄두어야 했던 작업들을 수행할 수 있습니다. 예들 들자면, 곧바로 보여게 될 수도 있는 이미지, 비디오, 다른 섹션의 내용을 호출하는 동작들입니다.

이후 사용자의 상호작용을 기다리는데 이때 1장에서 알아본 것처럼 16ms 미만의 시간 안에 처리해야 유려하게 보일 수 있습니다. 이때 opacity, transform을 사용할 경우 composite만 trigger 되기 때문에 다른 layout이나 paint가 발생하여 다시 렌더링을 해야 하는 것을 막을 수 있습니다.

즉, 요약하자면. 사용자에게 유려한 페이지를 제공하는 방법은 아래와 같습니다.

1. Load : 사용자에게 보이는 페이지는 1초 이내에 렌더링 되어서 나와야 한다.
2. Idle : Load의 시간을 맞추기 위해 뒤로 미루었던 작업이나 미리 해두어야 할 작업을 선택적으로 수행한다.
3. Animation : 사용자 상호작용에 대한 프레임 렌더링은 16ms(초당 60프레임이내로 제공되어야 한다.
4. Response : 사용자 입력에 대해 100ms 이내에 어떤 방식으로든 응답한다.

어떤 시점에서 무엇을 할 수 있는지, 언제 할 수 있는지를 알면 성능개선에 도움이 됩니다. 

### 3. Weapons of Jank Destruction

Chrome Dev Tools에서 **Performance 탭**에서 변화를 레코드 할 수 있습니다.

이는 초당 프레임 수와 프레임마다 어떤 작업이 포함되었는지 알려줍니다.

여기에는 단계별로 색상 코드가 존재합니다.

파란색은  {% hl_text blue %}Parse HTML{% endhl_text %}입니다.
</br>
보라색은 {% hl_text purple %}Recalculate style{% endhl_text %}과 {% hl_text purple %}Layout{% endhl_text %}입니다.
</br>
녹색은 {% hl_text green %}Paint{% endhl_text %}와 {% hl_text green %}Composite{% endhl_text %}입니다.

초당 60프레임이 넘어가는 작업을 확인하고 개선할 수 있습니다.
확인 전에 현재의 값을 측정하는 작업은 중요합니다.
만약 측정을 하지 않고 수정한다면 수정 후 실제로 얼마나 개선되었는지 알 수 없습니다. 
차이를 비교하기 위해 사전에 값을 측정해 놓는 것은 매우 중요합니다.

**Performance 탭**에서 빨간색 삼각형을 성능에 문제가 있을 수 있는 부분에서 경고해줍니다.

해당 영역을 클릭하면 어디에서 시간을 오래 소비하고 있는지 코드 위치까지 찾아볼 수 있습니다.
