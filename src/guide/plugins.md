---
title: 플러그인 
type: guide
order: 18
---

## 플러그인 만들기 

플러그인이라 하면 일반적으로 Vue 에 전역 함수를 추가하는 일을 한다. 플러그인에 대한 구체적인 제약사항이 있는 것은 아니지만 우리가 작성할 수 있는 플러그인은 몇가지 유형으로 분류해 볼 수 있다. 

1. 전역 함수와 속성(property)을 추가하는 플러그인 [vue-element](https://github.com/vuejs/vue-element)

2. 전역으로 쓸 수 있는 기능들(directives/filters/transitions)을 제공하는 플러그인 [vue-touch](https://github.com/vuejs/vue-touch)

3. 전역으로 믹스인들을 제공하는 플러그인 e.g. [vuex](https://github.com/vuejs/vuex)

4. Vue.prototype에 메쏘드를 추가해 Vue 인스턴스 메쏘드를 추가하는 플러그인 (?)

5. 자체 API 를 제공하는 라이브러리형 플러그인. 위의 기능들을 동시에 섞어서 제공하기도 한다. e.g. [vue-router](https://github.com/vuejs/vue-router)

Vue.js 플러그인은 반드시 `install` 메쏘드를 public 으로 공개해야 한다. 이 메쏘드는 `Vue` 생성자와 옵션을 인자로 받아서 호출된다 :

``` js
MyPlugin.install = function (Vue, options) {
  // 1. 전역 메쏘드나 property 를 추가한다. 
  Vue.myGlobalMethod = function () {
    // something logic ...
  }

  // 2. 전역 기능을 추가한다.
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // something logic ...
    }
    ...
  })

  // 3. 믹스인을 추가해서 컴포넌트 생성시 옵션으로 사용한다.
  Vue.mixin({
    created: function () {
      // something logic ...
    }
    ...
  })

  // 4. Vue 인스턴스 메쏘드를 추가한다. 
  Vue.prototype.$myMethod = function (options) {
    // something logic ...
  }
}
```

## 플러그인 사용하기 

 `Vue.use()` 전역 함수를 호출해주면, 플러그인을 사용할 수 있다 :

``` js
// calls `MyPlugin.install(Vue)`
Vue.use(MyPlugin)
```

두번째 인자로 옵션을 넘길 수 있다 :

``` js
Vue.use(MyPlugin, { someOption: true })
```

`Vue.use` 전역함수를 사용하면 플러그인 사용 호출이 중복되더라도 자동으로 걸러준다. 그래서 여기저기에서 여러번 사용을 선언하더라도 플러그인은 한개만 등록된다. 

Vue.js 에서 공식적으로 제공하는 `vue-router` 같은 플러그인의 경우  `Vue` 객체가 전역으로 선언이 되어 있으면 자동으로 `Vue.use()` 를 호출하도록 되어 있기 때문에 구지 호출하지 않더라도 사용할 수 있다. 하지만 CommonJS 같은 모듈 환경에서는 명시적으로 `Vue.use()`를 명시적으로 호출해 주어야 한다. 

``` js
// When using CommonJS via Browserify or Webpack
var Vue = require('vue')
var VueRouter = require('vue-router')

// Don't forget to call this
Vue.use(VueRouter)
```

[awesome-vue](https://github.com/vuejs/awesome-vue#libraries--plugins) 을 보면 여러 커뮤니티에서 기능한 플러그인과 라이브러리들이 많이 있으니 한번 살펴보기 바란다. 
