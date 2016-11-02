---
title: 라우팅 
type: guide
order: 21
---

## 공식적으로 지원하는 라우터

대부분의 SPA(단일 페이지 애플리케이션) 개발에는 공식적으로 지원하는 [vue-router library](https://github.com/vuejs/vue-router) 를 사용할 것을 권장한다. 더 자세히 알아보고 싶다면 vue-router 의 [documentation](http://vuejs.github.io/vue-router/) 문서를 참고하기 바란다.

## 간단한 라우팅 방법

라우터가 제공하는 다양한 기능을 사용할 필요가 없이 매우 간단한 라우팅 기능만 필요하다면, 아래와 같이 페이지 단위로 렌더링을 교체하는 방법을 사용해 구현할 수도 있다.

``` js
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }

const routes = {
  '/': Home,
  '/about': About
}

new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```

HTML5의 history API와 함께 사용하면, 간단하면서도 왠만큼 기능을 갖춘 라우터를 만들 수 있다. 실제로 어떻게 구현 하는지 보고 싶다면 [this example app](https://github.com/chrisvfritz/vue-2.0-simple-routing-example) 예제를 보기 바란다.

## 기타 다른 라우터와 함께 쓰는 방법

3rd 파티에서 제공하는 다른 라우터들이 있다. [Page.js](https://github.com/visionmedia/page.js), [Director](https://github.com/flatiron/director), [similarly easy](https://github.com/chrisvfritz/vue-2.0-simple-routing-example/compare/master...pagejs) 등등... 
이들 라우터를 어떻게 사용하면 되는지 힌트를 주기 위, Page.js와 함께 사용하는 방법을 [complete example](https://github.com/chrisvfritz/vue-2.0-simple-routing-example/tree/pagejs) 에 예제로 설명해 놓았다.