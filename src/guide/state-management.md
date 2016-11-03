---
title: 상태 관리 
type: guide
order: 22
---

## Official Flux-Like Implementation

대규모 어플리케이션을 개발하다보면 그 복잡성이 매우 빠른 속도로 증가하고 컴포넌트와 컴포넌트들 사이에서 얼어나는 작업에 따라 수많은 상태들이 존재하게 된다. 이를 관리하기 위해 Vue 에서는 [vuex](https://github.com/vuejs/vuex) 라는 모듈을 제공한다. vuex는 [Elm](https://en.wikipedia.org/wiki/Elm_%28programming_language%29) 에서 영감을 받아 만든 상태관리 라이브러리이다. [vue-devtools](https://github.com/vuejs/vue-devtools) 툴과도 연동하고, 거의 아무런 설정 없이 데이터를 시간축으로 접근할 수 있다. (??)

### React 를 개발해본 개발자라면

만약에 React를 경험해본 개발자라면, react 진영에서 Flux 를 구현한 [redux](https://github.com/reactjs/redux) 와 vuex가 어떻게 다른지 궁금할 것이다. Redux는 실제로 어떤 view 레이어에도 쓸 수 있게 만들어졌고, [simple bindings](https://github.com/egoist/revue) 예제에서 보여주는 것처럼 Vue 와 같이 써도 무방하다. Vuex가 다른 점이 있다면 vuex는 이것을 쓰는 앱이 Vue 앱이라는 사실을 _전제_하고 있다는 것이다. 그래서 Vue와 훨씬 쉽고 편하게 어울릴 수 있고 개발자들이 보기에 API가 좀 더 직관적이고 생산성 있게 만들어져 있다고 할 수 있다.

## 라이브러리 없이 간단하게 상태 관리하기

사람들이 쉽게 생각하고 넘어가는 거지만, Vue 애플리케이션이 다루는 진짜 핵심은 원시 `data` 객체이다. Vue 인스턴스는 단순히 이 데이터의 접근을 대리해주는 프락시 역할을 한다. 따라서 여러 Vue 인스턴스가 공유해야 할 상태값이 있다면 아이디 레퍼런스를 통해 쉽게 데이터를 공유할 수 있다. 

``` js
const sourceOfTruth = {}

const vmA = new Vue({
  data: sourceOfTruth
})

const vmB = new Vue({
  data: sourceOfTruth
})
```

`sourceOfTruth` 값이 변경될 때마다 `vmA` 와 `vmB` 컴포넌트는 각각의 뷰를 자동으로 어데이트 할 것이다. 이들의 하위 컴포넌트들도 데이터를 접근할때는 `this.$root.$data`로 접근을 한다. 그러면 한개의 객체로 모든 뷰가 동일하게 움직이게 된다. 하지만 이렇게 쓰면 디버깅이 힘들다. 어떤 컴포넌트가 데이터를 어느 타이밍에 접근해서 변경했는지 근거를 남기지 않기 때문에 알 수가 없게 된다. 

이 문제를 해결하기 위해 간단한 **store 패턴** 을 구상해 볼 수 있다 :

``` js
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    this.debug && console.log('setMessageAction triggered with', newValue)
    this.state.message = newValue
  },
  clearMessageAction () {
    this.debug && console.log('clearMessageAction triggered')
    this.state.message = 'action B triggered'
  }
}
```

store 의 상태를 변경시키려는 어떤 시도든 store를 거쳐야만 한다. 이렇게 상태관리를 한곳으로 집중시킴으로서 어떤 데이터 변경이 일어났고 어디서 요청이 발생했는지를 쉽게 이해하고 관리할 수 있다. 만약 잘못되면 로그를 통해 버그를 추적할 수 있다.

추가로 더 장치를 만들자면, 각각의 컴포넌트와 컴포넌트의 인스턴스들은 각각의 캡슐화된 상태값(state)를 따로 관리하도록 할 수 있다 : 

``` js
var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})
```

![State Management](/images/state.png)

<p class="tip">여기서 중요한 점은 절대로 상태값을 저장하는 객체를 다른 인스턴스 객체로 교체(대체: replace)하지 말아야 한다는 점이다. 각각의 Vue 객체들이 같은 상태 객체를 참조하고 있기 때문에 이를 교체하게 되면 감시자들이 이를 놓치게 된다.</p>

컴포넌트가 store에 저장된 상태값을 바로 수정하는 것을 막을 수 있는 장치를 계속해서 고민해본 결과 이벤트 발생을 통해 store 데이터를 수정해야 한다는 [Flux](https://facebook.github.io/flux/) 의 아키텍쳐와 동일한 결론에 도달했다. 이렇게 하면 store에 저장된 모든 상태 데이터의 변경을 기록할 수 있고 변경 로그를 통해 디버깅이 쉬워지며, 스냅샷과 변경이력을 시간순으로 재생할 수 있는등 많은 이점들이 생긴다.

이런 저런 고민을 해보면 역시 [vuex](https://github.com/vuejs/vuex) 을 사용하는 것이 좋겠다는 결론에 다다른다. 그러니 여기까지 읽었다면 망설이지 말고 지금 당장 써보시라!