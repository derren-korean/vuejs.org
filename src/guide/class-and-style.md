---
title: 클래스 / 스타일 바인딩
type: guide
order: 6
---

 데이터를 엘리먼트에 바인딩 할 경우, 엘리먼트의 클래스와 스타일 정보를 데이터에서 제공하는 경우가 많이 있다. 엘리먼트 입장에서는 클래스나 스타일 모두 어트리뷰트이기 때문에 `v-bind` 명령어 구문을 이용해 표현해 줄 수 있다. 기존 문법의 표현식을 이용해 결과값이 어트리뷰트 값으로 나오게 하면 된다. 하지만 문자열을 style 문법에 맞게 맞춰줘야 하고, 클래스를 표현하기 위해 if 구문을 이용해야 하는등, 복잡한 방법을 섞어 써야 하기 때문에 `v-bind`가 `class`나 `style` 어트리뷰트에 적용될때 한해 특별히 추가적인 방법을 제공한다. 이 경우 `문자열` 값 뿐만 아니라 `객체` 형태나 `배열` 형태의 값도 허용해서 처리해 준다.


## class 어트리뷰트 바인딩

### 값이 `객체`인 경우

`v-bind:class` 값으로 객체를 주면, 동적으로 class 를 적용/해제 할 수 있다.
``` html
<div v-bind:class="{ active: isActive }"></div>
```

 위 구문의 경우, 데이터의 `isActive` 값이 [true냐 false냐](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) 에 따라 active 클래스가 적용되거나 해제된다. (역자: 그렇다고 값이 항상 Boolean 타입일 필요는 없다. 일반적인 자바스크립트의 ture/false 판정법을 따른다.)

 객체 안에 여러개의 클래스를 주어 선택적으로 혹은 상황에 맞게 클래스를 적용시킬 수 있다. 또 동적으로 적용해야 하는 클래스와는 별도로 항상 적용해주어야 하는 클래스는 `v-bind:class` 대신 그냥 `class` 어트리뷰트에 값을 주면 되고, 이 둘은 같이 쓸 수 있다 :

``` html
<div class="static"
     v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>
```

데이터값이 아래와 같다면 :

``` js
data: {
  isActive: true,
  hasError: false
}
```

결과적으로 아래와 같은 html이 출력된다 :

``` html
<div class="static active"></div>
```

 `isActive` 나 `hasError` 값이 바뀌면, div 엘리먼트의 class 값도 동적으로 업데이트 된다. 예를 들어 `hasError` 값이 `true`가 되면, div 엘리먼트의 class 값은 `"static active text-danger"`이 된다.

class에 바인딩하는 객체를 반드시 리터럴 형태의 인라인 객체로 기술할 필요는 없다 :

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

data 의 classObject 객체를 주더라도 결과는 같다. 또한 [computed property](computed.html) 에 어트리뷰트 값을 바인딩 시키면 훨씬 강력하게 응용할 수 있다 :

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal',
    }
  }
}
```

### 값이 `배열`인 경우 

엘리먼트에 클래스를 적용하기 위해 `v-bind:class` 에 배열값을 줄 수 있다 :

``` html
<div v-bind:class="[activeClass, errorClass]">
```
``` js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

이 경우 결과는 :

``` html
<div class="active text-danger"></div>
```

배열 안에 조건에 따라 특정 클래스를 주거나 빼고 싶다면, 삼항 연산자를 써서 구현할 수도 있다 :

``` html
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
```

위 표현은 `errorClass`는 항상 적용이 되고, `activeClass` 클래스는 `isActive` 값이 `true` 인 경우에만 적용된다.

하지만 이런 식으로 삼항 연산자를 여러개 쓰다보면 표현이 난잡해 보일 수 있다. 그래서 배열 안에도 객체 표현을 넣을 수 있도록 허용해주었다 :

``` html
<div v-bind:class="[{ active: isActive }, errorClass]">
```

### 컴포넌트에서 사용할때

> This section assumes knowledge of [Vue Components](components.html). Feel free to skip it and come back later.

When you use the `class` attribute on a custom component, those classes will be added to the component's root element. Existing classes on this element will not be overwritten.

For example, if you declare this component:

``` js
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```

Then add some classes when using it:

``` html
<my-component class="baz boo"></my-component>
```

The rendered HTML will be:

``` html
<p class="foo bar baz boo">Hi</p>
```

The same is true for class bindings:

``` html
<my-component v-bind:class="{ active: isActive }"></my-component>
```

When `isActive` is truthy, the rendered HTML will be:

``` html
<div class="foo bar active"></div>
```

## style 어트리뷰트 바인딩

### `객체` 값인 경우

`v-bind:style` 값으로 객체를 주는 경우, 그 표현 방법이 매우 직관적이다. CSS 구문을 그대로 사용하되 자바스크립트 객체로만 표현해주면 된다. `style`의 속성 이름은 camelCase 도 허용되고 kebab-case 도 허용된다 :

``` html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
``` js
data: {
  activeColor: 'red',
  fontSize: 30
}
```

템플릿을 깔끔하게 하려면 데이터에 별도의 스타일 객체를 만들고 이 객체를 `style` 어트리뷰트에 바인딩 하는 것이 좋은 사용법이다 :

``` html
<div v-bind:style="styleObject"></div>
```
``` js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

마찬가지로, `computed` 값으로 스타일 객체를 주어 사용할 수도 있다. 

### `배열` 값인 경우 

 `v-bind:style` 값으로 배열을 사용하는 경우, 여러개의 스타일 값을 갖는 객체를 배열에 넣어서 넘겨줄 수 있다. 이 경우 스타일 값은 두 객체의 합집합으로 표현이 된다.

``` html
<div v-bind:style="[baseStyles, overridingStyles]">
```

### Auto-prefixing

각 벤더들마다 접두사를 제공하는 스타일 값을 `v-bind:style`에 사용해야 하는 경우 대표 스타일만 주면 벤더 접두사는 알아서 붙여준다. 예를 들어 `transform` 값을 스타일로 주면, Vue 는 자동으로 { -webkit-transform : , -ms-transform: , transform : } 값을 모두 적용해 준다.
