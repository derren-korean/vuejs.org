---
title: 사용자 정의 디렉티브(directive:명령어)
type: guide
order: 16
---

## 소개

코어 모듈에 내장된 기본 디렉티브들(`v-model`이나 `v-show` 처럼) 외에, 사용자가 직접 제작한 커스텀 디렉티브를 등록해서 사용할 수 있다. Vue 2.0에서 코드를 재사용하거나 추상화하는 장치는 누가 뭐라해도 컴포넌트여야하지만, 뷰단의 DOM 엘리먼트를 로우레벨에서 핸들링해야 하는 경우가 종종 있는데, 이 경우 사용자 정의 디렉티브는 여전히 유용하게 사용될 수 있다. 그 한 예가 input 엘리먼트에 포커스를 주는 것을 구현하는 것인데, 아래에 동작하고 있다 : 

{% raw %}
<div id="simplest-directive-example" class="demo">
  <input v-focus>
</div>
<script>
Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})
new Vue({
  el: '#simplest-directive-example'
})
</script>
{% endraw %}

페이지가 로드되면 input 태그는 포커스를 얻게 된다.  사실 이 페이지에 들어와서 다른 곳을 클릭하지 않았다면 위의 input 태그가 자동으로 포커싱 된다. 이제 이 디렉티브를 어떻게 구현했는지 살펴보자 : 

``` js
// v-focus라는 사용자 정의 디렉티브를 전역으로 선언한다
Vue.directive('focus', {
  // 디렉티브를 선언한 엘리먼트가 DOM에 붙으면 ...
  inserted: function (el) {
    // 엘리먼트에 포커스를 준다
    el.focus()
  }
})
```
만약 디렉티브를 전역이 아닌 로컬 스콥으로 등록하고 싶다면 컴포넌트 생성 옵션에 `directives` 를 주면 된다 :

``` js
directives: {
  focus: {
    // directive definition
  }
}
```

이렇게 선언한 다음 템플릿 어디에서든 엘리먼트에 `v-focus` 라는 디렉티브 어트리뷰트를 사용할 수 있다 :

``` html
<input v-focus>
```

## 후킹 함수들

디렉티브를 정의하는 객체에서 사용할 수 있는 후킹 함수들은 다음과 같다. (반드시 정의해야 하는 것은 아니고 선택적으로 정의하면 된다) :

- `bind`: 한번만 호출된다. 이 디렉티브가 선언된 엘리먼트에 최초로 묶일때(bound). 이 시점에 필요한 셋업을 해당 엘리먼트에 수행할 수 있다.

- `inserted`: 바운딩(묶인)된 엘리먼트가 부모 노드에 삽입된 직후 호출된다. (그렇기 때문에 이 함수가 호출될때는 부모 노드가 반드시 존재한다. 하지만 document 의 노드인 것을 보장하지는 않는다. 다시 말해 shadow dom의 일부분일 수도 있다).

- `update`: called after the containing component has updated, __but possibly before its children have updated__. The directive's value may or may not have changed, but you can skip unnecessary updates by comparing the binding's current and old values (see below on hook arguments).

- `componentUpdated`: 하위 컴포넌트와 __하위 노드__가 업데이트 된 이후 호출된다.

- `unbind`: 한번만 호출된다. 이 디렉티브가 엘리먼트에 묶여있는(bound)게 풀렸을때.

바로 다음에 이 후킹 함수들에 넘겨줄 인자들이 뭐가 있는지 살펴볼 것이다.(`el`, `binding`, `vnode`, `oldVnode` 같은 인자들)

## 후킹 함수에 넘겨주는 인자들

디렉티브의 후킹 함수들에 전달되는 인자들은 다음과 같다 :

- **el**: 디렉티브가 묶인(bounded) 엘리먼트. DOM 엘리먼트. 이 인자를 통해 직접 DOM을 수정한다.
- **binding**: 다음과 같은 속성을 갖는 객체이다
  - **name**: 디렉티브의 이름, 앞의 `v-` 접두사를 제외한 부분.
  - **value**: 디렉티브의 어트리뷰트 값으로 전달된 값. 예를들어 `v-my-directive="1 + 1"`라면 그 값은 `2` 이다.
  - **oldValue**: 이전 값, `update` 와 `componentUpdated` 후킹 함수에서만 유효하다. 이전 값이 현재값과 같더라도 호출은 된다.
  - **expression**: 바인딩 된 값을 스트링으로 표현한 것. 예를 들어 `v-my-directive="1 + 1"` 이라면  expression 값은 `"1 + 1"`이 된다.
  - **arg**: 디렉티브에 전달된 인자들. (없을 수도 있다) 예를 들어 `v-my-directive:foo`라면 인자는 `"foo"`가 된다.
  - **modifiers**: 변경자(modifier)를 갖는 객체, 예를들어, `v-my-directive.foo.bar`라면 변경자 객체는 `{ foo: true, bar: true }` 이 된다.
- **vnode**: Vue 컴파일러가 만들어낸 가상 노드. 가상 노드에 대해서 더 자세히 알아보려면, [VNode API](/api/#VNode-Interface) 를 참고하기 바란다.
- **oldVnode**: 이전의 가상 노드.  `update`와 `componentUpdated` 후킹 함수인 경우에 사용한다.

<p class="tip"> `el` 인자를 제외한 다른 모든 인자들은 읽기전용으로만 사용해야하며, 절대로 그 값을 수정하면 안된다. 후킹 함수들 사이에 데이터를 공유하고 싶다면, 엘리먼트의 [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) 어트리뷰트를 사용하는 것을 권고한다. </p>

이런 속성을 잘 표현해주는 사용자 정의 디렉티브들의 예이다 :

``` html
<div id="hook-arguments-example" v-demo:hello.a.b="message"></div>
```

``` js
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

{% raw %}
<div id="hook-arguments-example" v-demo:hello.a.b="message" class="demo"></div>
<script>
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})
new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
</script>
{% endraw %}

## 함수 간략표현 하기

많은 경우, `bind` 와 `update` 후킹 함수가 하는 일이 같은데, 이 경우 다른 후킹 함수들은 모두 생략하고 한개의 함수만 인자로 넘겨주면 된다. 예를 들어 : 

``` js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## 객체 리터럴들

만약 정의한 디렉티브가 여러개의 인자값이 필요하다면 자바스크립트 리터럴 표현으로 객체를 넘길 수 있다(이때 데이터는 binding.value로 접근한다) 이처럼 디렉티브는 어떤 자바스크립트 표현식도 값으로 갖을 수 있다.

``` html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```

``` js
Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```
