---
title: Render 함수 
type: guide
order: 15
---

## 기본적인 내용  

대부분의 경우 Vue 컴포넌트에서 HTML 은 template 에 작성할 것을 권장한다. 하지만 경우에 따라 HTML 렌더링을 자바스크립트로 해야 할 필요가 있는 경우도 생긴다. 이때가가 바로 **render  함수**가 필요한 때이다. 

구체적으로 `render` 함수를 쓰는게 더 실용적인 경우를 살펴보자. 예를 들어 링크가 걸린 타이틀(h1 ~ h6의 해딩태그)이 있다고 해보자.

``` html
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```

위와 같은 HTML을 만들기 위해 아래와 같이 사용하도록 컴포넌트 인터페이스를 설계했다고 하면 : 

``` html
<anchored-heading :level="1">Hello world!</anchored-heading>
```

`level` 이라는 속성에 따라 해딩태그의 수준을 결정해서 태그를 만들어야 하기 때문에 아래와 같은 구조를 떠올릴 것이다 : 

``` html
<script type="text/x-template" id="anchored-heading-template">
  <div>
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-if="level === 6">
      <slot></slot>
    </h6>
  </div>
</script>
```

``` js
Vue.component('anchored-heading', {
  template: '#anchored-heading-template',
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

그런데 이 템플릿은 그다지 좋아보이지 않는다. 'v-if'를 장황하게 늘여서 써야하는 것도 그렇고 똑같은 `<slot></slot>` 태그를 반복해서 써주는 것도 중복코드라고 봐야 한다. 게다가 컴포넌트의 최상위 엘리먼트는 두개 이상일 수 없기 때문에 불필요하게 `div` 태그로 감싸줘야 한다. 

템플릿을 쓰는것 자체는 훌륭한 방법이지만, 이 경우는 그렇게 훌륭한 경우가 아닌것이 분명하다. 그래서 대안으로 `render` 함수를 써서 다시 구현해 보았다. 

``` js
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // tag name
      this.$slots.default // array of children
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

훨씬 간단해 졌다. 코드도 짧아졌고 훨신 더 Vue 스럽게 만들어 졌다. 이 경우 `slot` 어트리뷰트가 없는 태그나 컨텐츠를 컴포넌트 안에 넣었다면, 예를 들어 위의 경우에는 `anchored-heading` 태그 안에 `Hello world!` 라는 텍스트만 넣었는데, 이 경우 이 텍스트가 `$slots.default` 에 저장이 되고, heading 태그 안에 들어가게 된다. 아직 이 내용을 잘 모른다면 ** render 함수에 대해 더 자세히 들어가기 전에 [instance properties API](/api/#vm-slots) 부분을 확인해 보기 바란다**

## `createElement` 인자

두번째로 렌더링에서 알아야 할 중요한 기능중에 하나가 `createElement` 라는 함수가 템플릿에서 어떻게 쓰이는지 알아야 하는 것이다. `createElement` 함수는 다음과 같은 인자를 받는다 : 

``` js
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // HTML 태그 이름, component options, or function
  // returning one of these. 반드시 와야 함.
  'div',

  // {Object}
  // 어트리뷰트 키-밸류 값을 갖는 객체
  // you would use in a template. 옵션 사항임.
  {
    // (see details in the next section below)
  },

  // {String | Array}
  // 자식이 될 VNode들. 옵션사항임.
  [
    createElement('h1', 'hello world'),
    createElement(MyComponent, {
      props: {
        someProp: 'foo'
      }
    }),
    'bar'
  ]
)
```

### Data 객체에 대한 팁 

어트리뷰트 바인딩을 설명할때 `v-bind:class` 나 `v-bind:style`의 경우 많이 쓰는 attribute이기 때문에 특별히 더 많은 장치를 제공했던 것처럼, root가 되는 Vue 객체의 data 필드도 그들만의 특별한 기능을 하는 특별 필드를 지원한다.

``` js
{
  // Same API as `v-bind:class`
  'class': {
    foo: true,
    bar: false
  },
  // Same API as `v-bind:style`
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // Normal HTML attributes
  attrs: {
    id: 'foo'
  },
  // Component props
  props: {
    myProp: 'bar'
  },
  // DOM properties
  domProps: {
    innerHTML: 'baz'
  },
  // Event handlers are nested under "on", though
  // modifiers such as in v-on:keyup.enter are not
  // supported. You'll have to manually check the
  // keyCode in the handler instead.
  on: {
    click: this.clickHandler
  },
  // For components only. Allows you to listen to
  // native events, rather than events emitted from
  // the component using vm.$emit.
  nativeOn: {
    click: this.nativeClickHandler
  },
  // Custom directives. Note that the binding's
  // oldValue cannot be set, as Vue keeps track
  // of it for you.
  directives: [
    {
      name: 'my-custom-directive',
      value: '2'
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // The name of a slot if the child of a component
  slot: 'name-of-slot'
  // Other special top-level properties
  key: 'myKey',
  ref: 'myRef'
}
```

### 재구성한 전체 예제 코드

이제까지의 내용을 바탕으로 위의 컴포넌트를 아래와 같이 작성해 볼 수 있다 : 

``` js
var getChildrenTextContent = function (children) {
  return children.map(function (node) {
    return node.children
      ? getChildrenTextContent(node.children)
      : node.text
  }).join('')
}

Vue.component('anchored-heading', {
  render: function (createElement) {
    // create kebabCase id
    var headingId = getChildrenTextContent(this.$slots.default)
      .toLowerCase()
      .replace(/\W+/g, '-')
      .replace(/(^\-|\-$)/g, '')

    return createElement(
      'h' + this.level,
      [
        createElement('a', {
          attrs: {
            name: headingId,
            href: '#' + headingId
          }
        }, this.$slots.default)
      ]
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

### 제약사항 

#### VNode(Virtual DOM에 만들어지는 노드)들은 중복되면 안된다(Unique)

컴포넌트 트리의 가상 DOM에 만들어지는 VNode 는 반드시 유일해야 한다. 다시말해 다음과 같은 render 함수는 유효하지 않다 : (VNode를 중복사용하고 있기 때문에 마지막 노드만 남을 것이다.)

``` js
render: function (createElement) {
  var myParagraphVNode = createElement('p', 'hi')
  return createElement('div', [
    // Yikes - duplicate VNodes!
    myParagraphVNode, myParagraphVNode
  ])
}
```

만약 같은 컴포넌트를 중복해서 여러개 만들어서 사용하고 싶다면, factory 함수를 사용하면 된다. 예를 들어 아래 예제와 같이 쓰면 20개의 동일한 문단(paragraph)을 무사히(valid) 뿌려낼 수 있다 : 

``` js
render: function (createElement) {
  return createElement('div',
    Array.apply(null, { length: 20 }).map(function () {
      return createElement('p', 'hi')
    })
  )
}
```

## 템플릿으로 작성하지 말고 자바스크립트로 해 보기

자바스크립트 만으로 간단하게 작업할 수 있다면 template 대신 render 함수를 써보는 것도 좋은 대안이 될 수 있다. 예를 들어, `v-if`와 `v-for`를 사용한 템플릿이 아래와 같다면 :

``` html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```

render 함수에 자바스크립트의 `if`/`else` 와 `map` 함수를 사용해서 구현할 수 있다. 

``` js
render: function (createElement) {
  if (this.items.length) {
    return createElement('ul', this.items.map(function (item) {
      return createElement('li', item.name)
    }))
  } else {
    return createElement('p', 'No items found.')
  }
}
```

## JSX

그렇다고 이것저것 다 `render` 함수로 작성하려면 매우 고통스러운 개발이 될 것이다 :

``` js
createElement(
  'anchored-heading', {
    props: {
      level: 1
    }
  }, [
    createElement('span', 'Hello'),
    ' world!'
  ]
)
```

특히나 html 구조가 별거 없다면 template 을 사용하는 편이 훨씬 간단하다.

``` html
<anchored-heading :level="1">
  <span>Hello</span> world!
</anchored-heading>
```

그래서 [Babel plugin](https://github.com/vuejs/babel-plugin-transform-vue-jsx) 로더를 두어 Vue 가 JSX를 사용하도록 하면 되는데, 템플릿을 사용할때와 거의 흡사하다 : 

``` js
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render (h) {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```

<p class="tip"> `createElement` 변수를 `h` 로 명명하는 것은 Vue 개발에서 자주 보게 될 관습화된 변수이다. JSX를 사용한다면 이 변수는 반드시 필수이다. 만약 변수명도 `h`로 해주어야 하는데, 만약 없으면 app이 에러를 던질 것이다.</p>

JSX 가 어떻게 자바스크립트에 어떻게 매핑되는지 알고 싶다면 [usage docs](https://github.com/vuejs/babel-plugin-transform-vue-jsx#usage) 를 참고하기 바란다

## Functional Components

The anchored heading component we created earlier is relatively simple. It doesn't manage any state, watch any state passed to it, and it has no lifecycle methods. Really, it's just a function with some props.

In cases like this, we can mark components as `functional`, which means that they're stateless (no `data`) and instanceless (no `this` context). A **functional component** looks like this:

``` js
Vue.component('my-component', {
  functional: true,
  // To compensate for the lack of an instance,
  // we are now provided a 2nd context argument.
  render: function (createElement, context) {
    // ...
  },
  // Props are optional
  props: {
    // ...
  }
})
```

Everything the component needs is passed through `context`, which is an object containing:

- `props`: An object of the provided props
- `children`: An array of the VNode children
- `slots`: A function returning a slots object
- `data`: The entire data object passed to the component
- `parent`: A reference to the parent component

After adding `functional: true`, updating the render function of our anchored heading component would simply require adding the `context` argument, updating `this.$slots.default` to `context.children`, then updating `this.level` to `context.props.level`.

Since functional components are just functions, they're much cheaper to render. They're also very useful as wrapper components. For example, when you need to:

- Programmatically choose one of several other components to delegate to
- Manipulate children, props, or data before passing them on to a child component

Here's an example of a `smart-list` component that delegates to more specific components, depending on the props passed to it:

``` js
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true,
  render: function (createElement, context) {
    function appropriateListComponent () {
      var items = context.props.items

      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,
      context.children
    )
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  }
})
```

### `slots()` 대 `children`

`slots()` 이 `children` 둘다 필요한 이유가 뭔지 궁금할지도 모르겠다. `slots().default`가 결과적으로 `children` 과 항상 같은것이 아닌가? 물론 그런 경우도 있지만, 아래와 같은 자식 노드들을 갖는 functional 컴포넌트가 있다면 어떨까?

``` html
<my-functional-component>
  <p slot="foo">
    first
  </p>
  <p>second</p>
</my-functional-component>
```

For this component, `children` will give you both paragraphs, `slots().default` will give you only the second, and `slots().foo` will give you only the first. Having both `children` and `slots()` therefore allows you to choose whether this component knows about a slot system or perhaps delegates that responsibility to another component by simply passing along `children`.
이경우, `children` 두개의 p 엘리먼트를 리턴하지만 `slots().default`는 두번째 p 태그만, 그리고 `slots().foo` 가 첫번째 p 태그를 리턴할 것이다.  `children` 과 `slots()`을 통해 우리는 이 컴포넌트가 slot 을 사용하는지 그렇지 않은지를 알 수 있다.(???)

## Template Compilation

You may be interested to know that Vue's templates actually compile to render functions. This is an implementation detail you usually don't need to know about, but if you'd like to see how specific template features are compiled, you may find it interesting. Below is a little demo using `Vue.compile` to live-compile a template string:

{% raw %}
<div id="vue-compile-demo" class="demo">
  <textarea v-model="templateText" rows="10"></textarea>
  <div v-if="typeof result === 'object'">
    <label>render:</label>
    <pre><code>{{ result.render }}</code></pre>
    <label>staticRenderFns:</label>
    <pre v-for="(fn, index) in result.staticRenderFns"><code>_m({{ index }}): {{ fn }}</code></pre>
  </div>
  <div v-else>
    <label>Compilation Error:</label>
    <pre><code>{{ result }}</code></pre>
  </div>
</div>
<script>
new Vue({
  el: '#vue-compile-demo',
  data: {
    templateText: '\
<div>\n\
  <h1>I\'m a template!</h1>\n\
  <p v-if="message">\n\
    {{ message }}\n\
  </p>\n\
  <p v-else>\n\
    No message.\n\
  </p>\n\
</div>\
    ',
  },
  computed: {
    result: function () {
      if (!this.templateText) {
        return 'Enter a valid template above'
      }
      try {
        var result = Vue.compile(this.templateText.replace(/\s{2,}/g, ''))
        return {
          render: this.formatFunction(result.render),
          staticRenderFns: result.staticRenderFns.map(this.formatFunction)
        }
      } catch (error) {
        return error.message
      }
    }
  },
  methods: {
    formatFunction: function (fn) {
      return fn.toString().replace(/(\{\n)(\S)/, '$1  $2')
    }
  }
})
console.error = function (error) {
  throw new Error(error)
}
</script>
<style>
#vue-compile-demo pre {
  padding: 10px;
  overflow-x: auto;
}
#vue-compile-demo code {
  white-space: pre;
  padding: 0
}
#vue-compile-demo textarea {
  width: 100%;

}
</style>
{% endraw %}
