---
title: MVC & 옵저버 패턴 적용하기
categories: 
  - js
  - pattern
tags: 
  - mvc
  - project
  - javascript
keywords:
  - mvc
  - project
  - javascript
thumbnailImage: thumbnailImage1.jpg
thumbnailImagePosition: bottom
metaAlignment: center
coverImage : coverImage.jpg 
date: 2018-09-22 12:29:20

---

ES6를 활용하여 MVC 및 옵저버 패턴을 적용한 프로젝트를 진행하면서 느꼈던 작업과 과정을 정리했습니다.

<!-- more -->

## 패턴의 필요성

돌이켜보면 가장 난감한 경우는 의존성이 고려되지 않은 프로젝트를 였습니다. 의존성이 고려되지 않고 작업 된 프로젝트는 유지보수에 많은 시간이 필요했습니다.

이에 못지않게 또 한 가지 어러운 일 중의 하나는 바로 데이터 변경에 따른 뷰 변경 작업이라고 생각합니다. 데이터가 많고 복잡할수록 뷰 또한 많은 변화가 있어 일일이 대응하기가 쉽지 않습니다.

그래서 이러한 문제들을 개선해보고자 MVC 및 옵저버 패턴을 도입해 해당 문제들을 개선한 프로젝트를 진행하기로 했습니다.

## 구조 설계하기

MVC 패턴은 담당하려는 역할별로 나누어서 각각에 대한 의존성을 낮출 수 있는 패턴입니다. 우선 패턴에 대한 학습 및 래펀런스로 아래 자료에서 도움을 많이 받았습니다.
> [코드스피츠75 - ES6+ 디자인패턴과 뷰패턴 #5](https://www.youtube.com/watch?v=LJhkPP0E6dw)

### Model의 역할
해당 기능에서 사용되는 데이터에 대한 참조는 모두 Model에서 담당합니다. Model은 다른 것들과 다르게 의존성이 전혀 존재하지 않는 독립적인 개체입니다. 

### View의 역할 
View는 Model 기반으로 렌더링할 DOM을 구축하는 역할을 맡습니다. 추가로 View에서 조금 더 복잡한 요구될 경우 하위에 `Template`을 두어서 복잡도를 낮추도록 하였습니다. DOM이 있기 때문에 직접 이벤트를 바인딩하는 부분도 포함됩니다.

### Controller의 역할
Controller는 View와 Model들에 접근하여 앱을 이어주는 단일 컨트롤러로 두었습니다. 초기 세팅, 기능 간의 로직 처리, View에 바인딩 될 이벤트 핸들러를 관리합니다.

## 싱글톤 처리

MVC 패턴에서 객체를 여러 개로 만드는 경우도 있고 하나만 만드는 경우도 있기 때문에 싱글톤 처리가 필요합니다. ES6에서 제공되는 코어객체 `WeackMap`를 사용해 싱글톤 객체를 생성합니다.

```javascript
// err = (v = 'invalid') => { throw v }
const Singleton = class extends WeakMap {
  has () { err() }
  get () { err() }
  set () { err() }
  getInstance (v) {
    if (!super.has(v.constructor)) super.set(v.constructor, v)
    return super.get(v.constructor)
  }
}
```

`WeackMap`은 객체를 키로 해서 값을 가질 수 있습니다. `WeackMap`에서 제공되는 has, get, set의 직접 사용을 막아서 `WeackMap`처럼 사용될 수 없게 합니다. getInstance()를 통해 키로 넘어온 객체의 생성자를 기준으로 사용하여 클래스당 인스턴스가 한 개씩 생성되도록 합니다.

Singleton 객체를 활용해서 Model, View, Controller의 에서 getInstance()를 활용해 싱글톤 객체를 얻을 수 있도록 합니다. 

```javascript
const singleton = new Singleton()
 const Controller = class {
  constructor (isSingleton) {
    if (isSingleton) return singleton.getInstance(this)
  }
  listen (model) {}
}
```

이 부분까지는 참고한 자료를 기반으로 적용할 수 있었습니다. 하지만 제가 작업하는 환경에서는 아래와 같은 에러가 발생하며 babel로 컴파일된 코드에는 동작하지 않았습니다.

{% alert danger %}
Uncaught TypeError: Constructor WeakMap requires 'new'
{% endalert %}

그래서 답을 찾다가 다음과 같이 설정 파일을 변경하였는데 정상적으로 동작했습니다. 서버로 사용하고 중인 node 버전을 targets으로 설정하면 에러가 발생하지 않았습니다.

```json
{
  "presets": [
    ["env", {
      "targets": {
        "node": "8.11"
      }
    }]
  ]
}
```

하지만 새로 추가한 코드 때문에 전체 코드가 관장되는 설정 파일을 수정할 수가 없었기에 다시 원래 설정으로 돌렸습니다.

```json
{
  "presets": ['env']
}
```

그래서 newWeakMap을 new로 생성한 인스턴스를 사용할 수 있는 클래스를 새로 만들어 사용하는 식으로 수정하였습니다.

```javascript
class newWeakMap {
  constructor (init) {
    this._wm = new WeakMap(init)
  }
  has (v) { return this._wm.has(v) }
  get (v) { return this._wm.get(v) }
  set (k, v) {
    this._wm.set(k, v)
    return this
  }
}

const Singleton = class extends newWeakMap {
 ...
}
```

덕분에 설정 파일 변경 없이 싱글톤 객체를 사용할 수 있었습니다.

## Model에 옵저버 패턴 적용

옵저버 패턴을 지원하기 위해 모델에서는 ES6의 `Set`을 상속받아서 사용했습니다.

```javascript
// is = (t, p) => t instanceof p
// err = (v = 'invalid') => { throw v }

const Model = class extends Set {
  constructor (isSingleton) {
    super()
    if (isSingleton) return singleton.getInstance(this)
  }
  add () { err() }
  delete () { err() }
  has () { err() }
  registerCtrl (v) {
    if (!is(v, Controller)) err()
    super.add(v)
  }
  unregisterCtrl (v) {
    if (!is(v, Controller)) err()
    super.delete(v)
  }
  notify (data) {
    this._s.forEach(v => v.listen(this, data))
  }
}
```

`Set`과 배열의 차이점은 중복검사를 할 필요가 없다는 점입니다. `Set`은 들어오는 값에 대하여 중복 값은 무시합니다.

기본적인 옵저버 패턴을 위해 Controller를 Set에 추가, 삭제하고 notify 할 수 있는 기능을 추가합니다.

`WeakMap`과 마찬가지로 같은 문제가 발생해서 새로운 'Set'을 생성하여 문제를 해결합니다.

```javascript
class newSet {
  constructor (init) {
    this._s = new Set(init)
  }
  add (v) { this._s.add(v) }
  delete (v) { this._s.delete(v) }
  has (v) { return this._s.has(v) }
}

const Model = class extends newSet {
  ...
}
```
## View에 render 구현

view는 렌더링을 담당하는 역할로 그 기능은 render 함수가 수행합니다.

주입받은 model과 해당 view를 렌더링할 위치를 주입받아 해당 위치에 DOM을 추가합니다.

controller에는 이벤트 핸들러가 있어서 필요한 DOM에 이벤트를 바인딩하여 사용합니다.

```javascript
class view extends View {

  constructor (controller, isSingleton) {
    super(controller, isSingleton)
  }

  render (model = err(), $selector) {
    if (!is(model, myModel)) err()
    const {_controller: ctrl} = this
    $selector.innnerHTML(myTemlate(model))
  }

}
```

## Controller 구현

```javascript

class controller extends Controller {
  constructor () {
    super()
  }

  $list () {
    const view = new ListView(this, true)
    const model = new ListModel(true)
    view.render(model, document.getElementById('#list'))
  }

  ...

  listen (model) {
    switch (true) {
      case is(model, listModel): return this.$list()
      ....
    }
  }

}

```

컨트롤러에는 notify에 대응하는 Listen 함수를 생성하였고, 그 안에는 맞는 모델에 맞게 다시 view를 그릴 수 있도록 설정합니다.

그리고 controller에는 이벤트 핸들러 및 서비스 로직이 추가됩니다.

## 알게된 점

많은 부분이 패턴을 적용하기 이전보다 편리한 점이 있었습니다. 역할별로 기능들을 나눈 덕분에 코드가 더 명확하고 심플해 졌다는 것을 느낄 수 있었습니다. 또한, 옵저버 패턴을 활용해서 Model 변경으로 View를 렌더링하면서 세세한 View 컨트롤이 없어서 너무나 편리했습니다.

하지만 마냥 좋은 점만 있던 건 아니었습니다. 모든 작업이 그렇듯이 예외적인 케이스들이 있어서 그런 예외 대응하면서 패턴이라는 구조에 맞게 작업을 하는 건 쉽지가 않았습니다.

제가 머릿속에 처음 구상된 MVC는 아래와 같은 형태였습니다.
{% image center https://velopert.com/wp-content/uploads/2016/04/MVC.png 출처 : https://velopert.com/1225 %}

하지만 단일 컨트롤러에서 여러 모델과 뷰를 관리하고 그것들이 서로서로를 참조하여 모델을 변경하는 부분에서는 아래와 그림과 같은 상황이었습니다. 
{% image center https://velopert.com/wp-content/uploads/2016/04/MVC2.png 출처 : https://velopert.com/1225 %}

제가 진행한 프로젝트 Model과 View의 쌍이 6개였는데 만약 더 큰 규모의 프로젝트였으면 이러한 문제에 대하여 충분히 대응이 없다면 그 프로젝트도 결과 유지보수가 쉬운 코드가 될 수는 없다는 생각이 들었습니다.

이번 프로젝트를 통해 패턴에 대한 장점을 익히게 되어서 좋았지만, React와 View 같은 UI 라이브러리들이 얼마나 잘 만들어지고 편리한지 또한 알게 되었던 것 같습니다.
