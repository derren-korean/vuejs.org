---
title: 이벤트 핸들링
type: guide
order: 9
---

## Event 에 반응하기 (listen)

 `v-on` 디렉티브를 이용해 DOM 이벤트를 감지하고 자동 호출이 되면 자바스크립트 메쏘드를 호출할 수 있다.

에를 들어 :

``` html
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```
``` js
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```

그 결과 :

{% raw %}
<div id="example-1" class="demo">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
<script>
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
</script>
{% endraw %}

## 이벤트 핸들링 메쏘드

많은 경우, 이벤트 핸들러 함수의 로직이 한줄로 실행하기는 어렵기 때문에 `v-on` 어트리뷰트 값으로 실행가능한 자바스크립트 코드를 주기는 사실상 어렵다. 그렇기 때문에 `v-on` 에 호출할 함수명이 값으로 올 수 있다.

예를 들어 :

``` html
<div id="example-2">
  <!-- `greet` is the name of a method defined below -->
  <button v-on:click="greet">Greet</button>
</div>
```

``` js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // define methods under the `methods` object
  methods: {
    greet: function (event) {
      // `this` inside methods points to the Vue instance
      alert('Hello ' + this.name + '!')
      // `event` is the native DOM event
      alert(event.target.tagName)
    }
  }
})

// you can invoke methods in JavaScript too
example2.greet() // -> 'Hello Vue.js!'
```

그 결과 :

{% raw %}
<div id="example-2" class="demo">
  <button v-on:click="greet">Greet</button>
</div>
<script>
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  methods: {
    greet: function (event) {
      alert('Hello ' + this.name + '!')
      alert(event.target.tagName)
    }
  }
})
</script>
{% endraw %}

## 인라인 핸들링 메쏘드

메쏘드 이름을 직접주는 대신, 자바스크립트로 메쏘드를 호출하는 구문을 사용할 수 있다 :

``` html
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
```
``` js
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
```

그 결과 :
{% raw %}
<div id="example-3" class="demo">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
<script>
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
</script>
{% endraw %}

또, 메쏘드 호출 구문에 인자로 DOM 이벤트 객체를 넘겨줄 수 있다. 이 이벤트 객체는 `$event` 라는 변수를 이용해 넘겨야 한다 :

``` html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">Submit</button>
```

``` js
// ...
methods: {
  warn: function (message, event) {
    // now we have access to the native event
    if (event) event.preventDefault()
    alert(message)
  }
}
```

## 이벤트 변경자 (modifier)

이벤트 핸들러에서 이벤트의 전달을 막기 위해 `event.preventDefault()` 나 `event.stopPropagation()` 호출은하는 것은 종종 사용하는 구문이다. 이런 구문을 메쏘드 안에서 코딩해 주는 것도 방법이지만, 메쏘드 안에서는 이벤트에 대한 후처리에 신경쓰지 않고 순수한 데이터 처리 로직만 담당한다면 더 좋을 것이다. 
이때문에 Vue 는 `v-on`에 ** event modifiers (이벤트 변경자)** 라는 구조를 두고 있다. 이 변경자는 이벤트의 디렉티브 뒤에 . 뒤에 오는데 다음과 같은 것들이 있다.

- `.stop`
- `.prevent`
- `.capture`
- `.self`

``` html
<!-- 클릭 이벤트 전달이 멈춘다 -->
<a v-on:click.stop="doThis"></a>

<!-- the submit event will no longer reload the page -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- modifiers can be chained -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 변경자만 쓸 수도 있다 -->
<form v-on:submit.prevent></form>

<!-- use capture mode when adding the event listener -->
<div v-on:click.capture="doThis">...</div>

<!-- only trigger handler if event.target is the element itself -->
<!-- i.e. not from a child element -->
<div v-on:click.self="doThat">...</div>
```

## 키 이벤트 변경자(modifier)

키보드 이벤트를 잡아낼 때는 일반적인 키 코드 값을 직접 써주면 된다 :

``` html
<!-- only call vm.submit() when the keyCode is 13 -->
<input v-on:keyup.13="submit">
```

키코드 값을 모두 숫자로 기억하기는 어려운 일이다. 그래서 일반적으로 많이 쓰는 alias 를 대신 사용하는 것을 지원한다 :

``` html
<!-- same as above -->
<input v-on:keyup.enter="submit">

<!-- also works for shorthand -->
<input @keyup.enter="submit">
```

키 변경자의 모든 리스트는 아래와 같다 :

- enter
- tab
- delete (captures both "Delete" and "Backspace" keys)
- esc
- space
- up
- down
- left
- right

전역 객체인 `config.keyCodes` 파일에 [사용자 정의 키코드 alias를 정의할 수 있다](/api/#keyCodes) :

``` js
// enable v-on:keyup.f1
Vue.config.keyCodes.f1 = 112
```

## Why Listeners in HTML?

You might be concerned that this whole event listening approach violates the good old rules about "separation of concerns". Rest assured - since all Vue handler functions and expressions are strictly bound to the ViewModel that's handling the current view, it won't cause any maintenance difficulty. In fact, there are several benefits in using `v-on`:

1. It's easier to locate the handler function implementations within your JS code by simply skimming the HTML template.

2. Since you don't have to manually attach event listeners in JS, your ViewModel code can be pure logic and DOM-free. This makes it easier to test.

3. When a ViewModel is destroyed, all event listeners are automatically removed. You don't need to worry about cleaning it up yourself.
