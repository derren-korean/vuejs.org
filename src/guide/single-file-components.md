---
title: 컴포넌트를 파일(뷰 파일) 하나로 작성하기 
type: guide
order: 19
---

## 소개 

많은 Vue 프로젝트에서 컴포넌트를 쓸때 `Vue.component` 를 이용해 컴포넌트를 전역으로 선언한 다음 웹 페이지의 body에 컨테이너가 되는 엘리먼트를 만들고 `new Vue({ el: '#container '})` 를 써서 구현할 것이다. 

상대적으로 작은 규모의 프로젝트에서는 뷰 몇개만 바꿔가면서 구현하면 되기 때문에 그다지 문제없이 동작할 것이다. 하지만 뷰가 많아지고 복잡해지는 프로젝트라면, 혹은 당신의 동료 개발자가 CSS나 HTML 을 등한시 하고 오직 자바스크립트로만 모든걸 해결하는 스타일이라면 다음과 같은 문제점들이 나타날 것이다 : 

- 모든 컴포넌트를 **전역 컴포넌트로 정의** 하면 컴포넌트 이름이 충돌할 가능성이 있다
- 컴포넌트에서 정의하는 **String templates** 에 HTML 을 작성하면 String 이기 때문에 태그 하이라이팅이 안되기 때문에 HTML 문법에 잘 맞지 않거나 태그를 실수할 수 있다.
- **컴포넌트에 CSS 지원이 안되면** HTML 과 JavaScript 만 컴포넌트 모듈에 사용되기 때문에 절름발이가 된다. CSS는 외부에 따로 존재해야 한다. 
- **빌드 단계를 따로 두지 않으면** 순수한 HTML과 자바스크립트 ES5 문법으로만 컴포넌트를 작성해야 한다. HTML을 작성하는 보다 훌륭한 처리기인 Pug(혹은 Jade)나 Babel을 이용해 진보한 자바스크립트를 쓸 수 있으면 좋겠는데...

이런 문제점들이 `.vue` 확장자를 사용한 ** vue 컴포넌트 파일 ** 을 사용해 개발하고 webpack 이나 browserify 같은 빌드 툴을 사용해 빌드한다면 한방에 해결된다.

아래 뷰파일로 만든 `Hello.vue` 예제가 있다 :

<img src="/images/vue-component.png" style="display: block; margin: 30px auto">

그래서 다음과 같은 이득을 었었다 :

- [HTML, CSS, 자바스크립트 모두 하이라이팅이 지원된다.](https://github.com/vuejs/awesome-vue#syntax-highlighting)
- [CommonJS 문법을 활용할 수 있다](https://webpack.github.io/docs/commonjs.html)
- [CSS 스콥을 컴포넌트에 한정할 수 있다](https://github.com/vuejs/vue-loader/blob/master/docs/en/features/scoped-css.md)

또 앞서 약속한 것처럼, Jade나 Babel, Stylus 같은 편리한 방법을 사용해 개발할 수 있다.

<img src="/images/vue-component-with-preprocessors.png" style="display: block; margin: 30px auto">

이 예제에서는 jade 와 CommonJS, stylus 를 사용해 컴포넌트를 작성했다. 그 밖에 툴에서 지원하는 로더가 있다면, Buble이나 TypeScript, SCSS, PostCSS 같은 다양한 언어들을 같이 사용할 수 있다.

<!-- TODO: include CSS modules once it's supported in vue-loader 9.x -->

## 시작하기 

### 자바스크립트로 모듈 시스템을 만드는데 익숙하지 않은 개발자라면 

`.vue` 컴포넌트 파일로 작업을 하려면 고급 자바스크립트 애플리케이션의 세계로 들어와야 한다. 그렇기 때문에 몇가지 새로운 개발 툴을 익혀야 한다.

- **노트 패키지 매니저 (NPM)**: [Getting Started guide](https://docs.npmjs.com/) 가이드의 _10: Uninstalling global packages_ 정도 읽어주면 좋다.

- **모던 자바스크립트 ES2015/16**: Babel 가이드를 읽어라 [Learn ES2015 guide](https://babeljs.io/docs/learn-es2015/). 모든 기능을 기억해야 할 필요는 없겠지만, 필요할때 찾아서 읽을 수 있으면 도움을 많이 받을 것이다. 

이 기술들을 깊이 공부한 다음에, [webpack-simple](https://github.com/vuejs-templates/webpack-simple) 템플릿도 자세히 보기를 권한다. 해보라는 데로 하면서 Vue 프로젝트를 `.vue` 컴포넌트와 핫 릴로딩 기술(수정하면 바로 ES2015 스크립트로 변환해서 브라우저에까지 반영이 되는)로 만들어보아야 한다.

템플릿에서는 [Webpack](https://webpack.github.io/) 기술을 활용하고 있는데, 웹팩은 여러 모듈을 애플리케이션에서 사용하기 편하도록 하나의 번들로 묶어주는 역할을 한다. 웹팩만 더 배워보고 싶다면 [this video](https://www.youtube.com/watch?v=WQue1AN93YU) 에 잘 소개되어 있다. 기본적인 내용을 익혔으면 [this advanced Webpack course on Egghead.io](https://egghead.io/courses/using-webpack-for-production-javascript-applications) 을 보기 바란다. 

웹팩은 "loader" 를 이용해 각 모듈을 변환해서 번들에 묶어낸다. Vue 는 `.vue` 확장자의 컴포넌트 파일을 변환할 수 있는 [vue-loader](https://github.com/vuejs/vue-loader) 플러그인을 제공한다. [webpack-simple](https://github.com/vuejs-templates/webpack-simple) 템플릿을 이용하면 모든 것이 설정되어 있다. 그래도 `.vue` 컴포넌트와 웹팩이 어떻게 동작하는 것인지 좀 더 알아보고 싶다면 [the docs for vue-loader](https://vue-loader.vuejs.org) 를 읽어보면 된다.

### 모듈 개발에 능숙한 개발자 라면 

Webpack 이든 Browserify 든 선호하는 것이 있다면, 어떤 규모의 프로젝이든 우리가 만들어놓은 템플릿이 있다. [github.com/vuejs-templates](https://github.com/vuejs-templates) 에 만들어 둔 템플릿들을 한번 훑어보고 적당한 경우의 템플릿을 선택해서 사용하기 바란다. 그리고 README 파일에 나온 설명을 따라 [vue-cli](https://github.com/vuejs/vue-cli) 를 활용해서 새 프로젝트를 만들고 시작하면 좋겠다.

