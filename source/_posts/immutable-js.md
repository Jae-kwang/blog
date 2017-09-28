---
title: immutable.js 알아보기
categories: javascript
tags:
  - Javascript
  - immutable.js
keywords:
  - Javascript
  - library
date: 2017-06-11 15:30:30
---

### Immutable.js 란? 

React를 접했을 때 immutable.js도 같이 알 되었는데, 당시에는 정확히 왜 필요한지 몰랐습니다. 
하지만 몇 번 보다 보니 그 이유를 알게 되었고, 제가 이해한 부분과 사용법을 정리해보려고 합니다.

<!-- more -->

#### 객체의 불변성

처음 'immutable:불변성'이란 단어는 낯설고 머리에 잘 와닿지 않았습니다.
불변성...불변성... 발음도 힘듭니다. 😧

저는 머리가 좋지 않은 관계로 한번은 더 풀어서 이해해야 했습니다. 

> 내가 만드는 객체는 절대 변하지 않아, 그러니까 수정하려면 반드시 다른 걸 하나 새로 만들어야 해!

React에서는 state 혹은 props에 변화가 감지되면 컴포넌트를 리랜더링합니다.

{% hl_text danger %}
그런데 레퍼런스 값으로 데이터를 가지고 있는 정보들은 값이 달라져도, 레퍼런스가 같기 때문에 React는 값의 변화를 인지하지 못합니다.
{% endhl_text %}

그래서 객체를 새로 만들어주어야 하는데 이때 immutable.js을 통해 더욱 손쉽게 데이터를 변경할 수 있습니다.

#### Without immutable.js

우선 immutable.js 없이 값을 수정하는 예제를 한번 확인해보시죠.

React에서는 위에도 언급했듯 레퍼런스를 활용하는 데이터의 경우에는 새로운 값을 만들어 연결해주어야 하는데, 다음 예제와 같이 데이터가 배열인 경우 특정 idx 내부의 값을 변경할 때는 아래와 같이 적용해 볼 수 있습니다.

```javascript
Items: [
    ...Items.slice(0, idx),
    {
      ...item,
      tempValue: !item.tempValue
    },
    ...Items.slice(idx + 1, Items.length)
]
```

1. 기존 배열에서 slice()를 사용해 idx까지 배열을 자른다.
2. 수정할 idx의 데이터를 수정한다.
3. 수정한 idx 이후에 데이터를 slice() 통해 자른다.
4. 3개의 작업 된 데이터를 하나의 배열을 만들어 다시 기존 변수에 넣어준다.

{% alert warning %}
단순히 값을 변경해주는 부분인데 매우 번거로울 뿐만 아니라, 데이터가 구조가 복잡하거나 많을 때는 더욱 많은 리소스 사용하게 될 것입니다.
{% endalert %}


```javascript
Items[idx].tempValue = !Items[idx].tempValue;
```

이렇게 사용해도 정상작동되면 참 좋을 텐데요? 😏


#### Use immutable.js

immutable 코드를 쓰기 전엔 8줄의 코드가 3줄로 줄었습니다.

```javascript
Items = Itmes.update(idx, (item) => {
   return item.set('tempValue', !item.get('tempvalue')); 
});
```

간단히 설명하면, immutable.js에서 제공되는 {% hl_text orange %}update{% endhl_text %}를 활용하여 변경하고자 하는 idx를 받아 해당 위치의 값을 {% hl_text orange %}set{% endhl_text %}로 수정하는데{% hl_text orange %}get{% endhl_text %}를 사용해 현재 값을 받아 변경합니다.
{% alert info %}
게다가 더 빠르다고 합니다.
http://blog.klipse.tech/javascript/2016/06/23/immutable-perf.html
{% endalert %}
### immutable.js 간단 사용법

##### 1. 데이터 생성

immutable을 사용하면 객체는 MAP으로 감싸주어야 합니다.
배열의 경우에는 List로 감싸줍니다.

```javascript
// basic object
var data = {
  a: 1,
  b: 2,
  c: {
    d: 3,
    e: 4,
    f: 5
  }
}

// use immutable.js
var Map = Immutable.Map;
var data = Map({
  a: 1,
  b: 2,
  c: Map({
    d: 3,
    e: 4,
    f: 5
  })
})
```



##### 2. 데이터 읽기

```javascript
/* 자바스크립트 객체로 변환하기 */
data.toJS(); // { a:1, b:2, c: { d: 3, e: 4, f: 5 } }

/* 특정 키의 값 얻어오기 */
data.get('a'); // 1

/* 내부의 키의 값 얻어오기 */
data.getIn(['c', 'd']) // 3
```

##### 3. 데이터 수정

```javascript
/* 값 설정하기 */
var newData = data.set('a', 4);

/* 내부의 값 설정하기 */
var newData = data.setIn(['c', 'd'], 10);

/* 내부의 객체를 설정하기 */
var newData = data.set('c', Map({ d: 10, e: 10, f: 10 }))

/* 값 여러개 설정하기 */
var newData = data.merge({ a: 10, b: 10 })

/* nest된 키에 값 여러개 설정하기 */
var newData = data.mergeIn(['c'], { d: 10, e: 10 });

/* List 일경우 값 추가하기 */
var newList = list.push(Map({value: 3}))

/* List 일경우 값 앞에 추가하기 */
var newList = list.unshift(Map({value: 0}))
```

##### 4. 데이터 삭제 

```javascript
/* 아이템 제거 */
var newList = list.delete(1);

/* 가장 마지막 아이템 제거 */
var newList = list.pop();
```

공식 페이지 : https://facebook.github.io/immutable-js/






