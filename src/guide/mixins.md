---
title: 믹스인 
type: guide
order: 17
---

## 기본적인 내용 

믹스인은 뷰 컴포넌트들이 재사용 가능한 기능들을 여러 뷰에서 사용할 수 있도록 하기 위한 장치이다. 믹스인 객체는 컴포넌트 생성자 옵션에서 갖는 모든 필드를 갖을 수 있다. 컴포넌트가 믹스인을 사용하면 믹스인에 있는 모든 옵션들이 컴포넌트의 옵션과 서로 섞이게(믹스) 된다.

Example:

``` js
// 믹스인 객체를 정의하고 
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 이 믹스인을 사용하는 컴포넌트를 정의한다 
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // -> "hello from mixin!"
```

## 옵션 머지(합치기) 

When a mixin and the component itself contain overlapping options, they will be "merged" using appropriate strategies. For example, hook functions with the same name are merged into an array so that all of them will be called. In addition, mixin hooks will be called **before** the component's own hooks:

``` js
var mixin = {
  created: function () {
    console.log('mixin hook called')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('component hook called')
  }
})

// -> "mixin hook called"
// -> "component hook called"
```

Options that expect object values, for example `methods`, `components` and `directives`, will be merged into the same object. The component's options will take priority when there are conflicting keys in these objects:

``` js
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})

vm.foo() // -> "foo"
vm.bar() // -> "bar"
vm.conflicting() // -> "from self"
```

Note that the same merge strategies are used in `Vue.extend()`.

## 전역 믹스인 (Global Mixin)

You can also apply a mixin globally. Use caution! Once you apply a mixin globally, it will affect **every** Vue instance created afterwards. When used properly, this can be used to inject processing logic for custom options:

``` js
// inject a handler for `myOption` custom option
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// -> "hello!"
```

<p class="tip">Use global mixins sparsely and carefully, because it affects every single Vue instance created, including third party components. In most cases, you should only use it for custom option handling like demonstrated in the example above. It's also a good idea to ship them as [Plugins](/guide/plugins.html) to avoid duplicate application.</p>

## Custom Option Merge Strategies

When custom options are merged, they use the default strategy, which simply overwrites the existing value. If you want a custom option to be merged using custom logic, you need to attach a function to `Vue.config.optionMergeStrategies`:

``` js
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // return mergedVal
}
```

For most object-based options, you can simply use the same strategy used by `methods`:

``` js
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```

A more advanced example can be found on [Vuex](https://github.com/vuejs/vuex)'s 1.x merging strategy:

``` js
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```
