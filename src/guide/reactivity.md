---
title: 반응성에 대해 좀 더 깊이 알아보기
type: guide
order: 12
---


기본적인 것들은 충분히 설명했다. 이제 좀 더 깊이 들어가 보자. Vue 에서 가장 눈에 띄는 특징은 억지스럽지 않고 기존 사용성에 그대로 동화되어 사용할 수 있는(unobtrusive) 반응형 시스템이라는 점이다. Model 은 그저 자바스크립트 객체일 뿐이다. 이 객체 값을 변경하면 뷰가 업데이트 된다. 그래서 데이터 관리가 매우 간단하고 직관적이다. 하지만 일반적인 오작동을 피하려면 이런 것들이 어떻게 동작하는지를 반드시 이해해야만 한다. 이번 장에서 Vue의 반응형 시스템이 어떻게 동작하는지 좀 더 낮은 레벨(lower level)에서 상세하게 들어가 볼 것이다.

## data 내용이 변경되는 것을 어떻게 알아내나

기본 JavaScript 객체를 `data` 옵션으로 Vue 인스턴스에 전달할 때 Vue.js은 그 모든 속성들을 일일이 돌아다니며  [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)을 사용하여 getter/setter로 변환한다. 이것은 ES5에서 새로 등장한 기능으로 하위호환이 불가능하기 때문에, Vue.js는 IE8 이하를 지원하지 않는다.

 getter/setters 는 사용자에게 보이지 않지만, 내부적으로 Vue 가 각 데이터 필드와 UI간의 의존성을 추적하거나 변경된 사실을 UI나 메쏘드에 알려주기 위한 용도로 활용된다. 한가지 아쉬운 점은 브라우저의 콘솔이 변환된 `data` 객체를 콘솔에 찍을때 getter/setter 를 지저분하게 보여준다는 점이다. 그래서 좀 더 편하게 `data` 객체를 보려면 [vue-devtools](https://github.com/vuejs/vue-devtools) 툴을 설치해서 보기를 권한다. 

모든 컴포넌트의 인스턴스는 각각 **watcher** 라는 객체를 갖고 있는데, 이 객체는 컴포넌트가 렌더링 될때 렌더링된 객체가 `data`의 어떤 필드에 의존한다고 판단되면 "touched" 라는 변수에 기록해 둔다. 이후 setter 함수가 호출이 되어 이 값이 변경되면 `watcher`에게 자동 통보(trigger) 되어 해당 뷰를 바로 업데이트 한다.

![Reactivity Cycle](/images/data.png)

## 변경된 것을 감지할때 주의해야 할 점들

모던 자바스크립트(`Object.observe` 기능을 빼버렸기 때문에...)의 한계 때문에, 뷰는 **property 가 추가되거나 삭제되는 것을 감지하지 못한다**. Vue 는 컴포넌트 인스턴스가 초기화 되는 시점에 getter/setter 변환 작업을 수행하기 때문에, 처음부터 `data` 에 해당 property가 있어야지 반응형 기능들을 장치시켜줄 수 있다. 예를 들어 :  

``` js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 은 반응형이 된다.

vm.b = 2
// `vm.b` 은 반응형이 안된다.
```

Vue 는 이미 생성된 인스턴스의 루트 `data` 에 동적으로 새로운 반응형 속성을 추가할 수 없다. 하지만, `Vue.set(object, key, value)` 메쏘드를 이용하면 그 하위 인스턴스들에는 동적으로 반응형 property를 넣어줄 수 있다. 

``` js
Vue.set(vm.someObject, 'b', 2)
```

아니면 뷰 인스턴스에서 직접 `vm.$set` 메쏘드를 호출해도 좋다. 이 메쏘드는 단순히 `Vue.set` 전역 함수를 호출해 주는 wrapper 메쏘드이다:

``` js
this.$set(this.someObject, 'b', 2)
```

 이미 존재하는 속성의 하위에 여러 속성(property)을 넣어 주어야 할 경우, `Object.assign()` 이나 `_.extend()`을 쓰면 안된다. 이렇게 추가된 속성은 getter / setter 가 붙지 않는다. 이런 경우에는 완전히 빈 객체를 만들고, 그 객체에 현재 객체와 덧붙이려는 속성을 더해서 추가한 다음 완전히 새로운 객체 참조값으로 세팅해 주어야 해당 속성의 하위 값까지 반응형이 된다. 

``` js
// instead of `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```

배열과 관련된 주의사항들도 있다. 이것은 이미 [리스트 나타내기 섹션](/guide/list.html#Caveats) 에서 이미 설명했다.

## 반응형 속성 선언하기

Vue 가 최상위 Vue 객체의 `data` 에 동적인 속성 추가를 금지하고 있기 때문에 Vue 인스턴스를 초기화 할때 반드시 반응형 속성에 대한 데이터 초기화를 선언해 주어야만 한다. 필요하다면 빈 문자열로라도 초기화를 해주어야 한다.

``` js
var vm = new Vue({
  data: {
    // declare message with an empty value
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// set `message` later
vm.message = 'Hello!'
```

위 예제에서, 만약 `message` 라는 속성을 선언하지 않았다면, Vue 는 존재하지 않는 속성에 접근하려 한다는 경고를 출력할 것이다. 

이런 제약사항을 둔 데는 기술적인 이유가 있다. 데이터 값이 변경되는 것을 감지하는 watcher 시스템이 해당 속성이 없어서 동작하지 않도록 하는 것을 방지할 수 있고 Vue 인스턴스의 데이터 타입 체크 시스템도 훨씬 매끄럽게 동작한다. 또한 코드의 유지보수성 측면에서도 이 점은 상당히 중요한 기여를 한다. `data` 객체는 컴포넌트의 상태를 정의하는 스키마와 같은 기능을 한다. 사전에 모든 반응형 속성들을 선언해 놓음으로서 컴포넌트에 대한 코드 이해와 가독성, 그리고 유지보수성이 훨씬 좋아진다고 본다. 


## 비동기 Update 큐 

아직 눈치채지 못했을지도 모르겠지만, Vue 는 DOM 을 **비동기적으로** 처리한다. 데이터 변경이 관측될 때마다 동일한 이벤트 처리 루프내에서 발생하는 모든 데이터 변경은 단일 큐와 버퍼에 쌓인다. 만약 같은 watcher 가 여러번 호출이 되더라도 하나의 큐에는 하나의 변경값만 남게 된다. 이 점은 불필요한 연산과 DOM조작을 피해갈 수 있다는 점에서 매우 중요하다. 그리고 나서 다음 이벤트 루프인 "tick" 이 호출되면 Vue는 큐에 쌓여 있는 데이터 변경에 따른 렌더링 작업을 수행한다(이미 중복 잡업이 배제된). 내부적으로 Vue 는 이 것을 처리할때 `Promise.then`과  `MutationObserver` 을 걸어두었다가 `setTimeout(fn, 0)` 을 호출해서 이것들을 처리한다. 

For example, when you set `vm.someData = 'new value'`, the component will not re-render immediately. It will update in the next "tick", when the queue is flushed. Most of the time we don't need to care about this, but it can be tricky when you want to do something that depends on the post-update DOM state. Although Vue.js generally encourages developers to think in a "data-driven" fashion and avoid touching the DOM directly, sometimes it might be necessary to get your hands dirty. In order to wait until Vue.js has finished updating the DOM after a data change, you can use `Vue.nextTick(callback)` immediately after the data is changed. The callback will be called after the DOM has been updated. For example:


``` html
<div id="example">{{ message }}</div>
```

``` js
var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'new message' // change data
vm.$el.textContent === 'new message' // false
Vue.nextTick(function () {
  vm.$el.textContent === 'new message' // true
})
```

There is also the `vm.$nextTick()` instance method, which is especially handy inside components, because it doesn't need global `Vue` and its callback's `this` context will be automatically bound to the current Vue instance:

``` js
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => 'not updated'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => 'updated'
      })
    }
  }
})
```
