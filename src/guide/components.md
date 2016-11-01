---
title: 컴포넌트 
type: guide
order: 11
---

## Component 란 무엇인가?

컴포넌트는 Vue의 가장 강력한 기능 중 하나이다. 컴포넌트는 기존의 HTML 엘리먼트로 만든 HTML fragment를 재사용 가능한 덩어리로 확장해 사용할 수 있게 해준다. 좀 더 고상하게 말하자면, 컴포넌트는 뷰(vue) 컴파일러가 추가해주는 메쏘드(behavior)를 단 `custom element(확장 엘리먼트)` 라고 볼 수 있다. 만약 custom element 를 쓰는데 거부감이 있다면, 기존 HTML 엘리먼트를 그대로 쓰고, `is` 어트리뷰트로 확장 엘리먼트를  마킹만 해주어도 된다.

## 컴포넌트 사용하기 

### 등록하기 

이전 섹션에서 `Vue.extend()`를 사용하여 컴포넌트 생성자를 만드는 방법을 알아보았다 :

``` js
new Vue({
  el: '#some-element',
  // options
})
```

전역 컴포넌트로 등록하려면, `Vue.component(tagName, options)`를 사용해서 등록한다. 예를 들어:

``` js
Vue.component('my-component', {
  // options
})
```

<p class="tip"> 비록 [W3C rules](http://www.w3.org/TR/custom-elements/#concepts)에서 제시하고 있는 태그 이름 규칙(모두 소문자이고, 반드시 - 을 포함해야 한다.)을 따르는 것이 바람직하다고 보지만, Vue 에서는 이를 강제하지는 않는다. </p>

일단 한번 등록되면, 컴포넌트는 컴포넌트의 인스턴스들의 템플릿에서 확장 엘리먼트(custom element)로 사용할 수 있다. `<my-component></my-component>`. 반드시 루트 Vue 인스턴스가 **생성되기 전**에 확장 엘리먼트가 등록되어 있어야 한다. 아래 전체 예제가 나온다.:

``` html
<div id="example">
  <my-component></my-component>
</div>
```

``` js
// register
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})

// create a root instance
new Vue({
  el: '#example'
})
```

이렇게 렌더링 된다.:

``` html
<div id="example">
  <div>A custom component!</div>
</div>
```

{% raw %}
<div id="example" class="demo">
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})
new Vue({ el: '#example' })
</script>
{% endraw %}

### 지역(local) 범위로 등록하기

You don't have to register every component globally. You can make a component available only in the scope of another instance/component by registering it with the `components` instance option:
모든 컴포넌트를 전역으로 등록해야 하는 건 아니다. 다른 컴포넌트 인스턴스의 `components` 생성 옵션에 등록하는 것으로, 그 컴포넌트의 범위 내에서만 사용할 수 있는 컴포넌트를 만들 수 있다:

``` js
var Child = {
  template: '<div>A custom component!</div>'
}

new Vue({
  // ...
  components: {
    // <my-component> 라는 확장 태그는 이 컴포넌트의 하위 컴포넌트들에서만 사용할 수 있다.
    'my-component': Child
  }
})
```

명령어(directive) 들 같이 등록해서 사용하는 Vue 의 기능들은 모두 이와같은 캡슐화 전략을 따른다. 

### DOM 템플릿 파싱시 주의사항

When using the DOM as your template (e.g. using the `el` option to mount an element with existing content), you will be subject to some restrictions that are inherent to how HTML works, because Vue can only retrieve the template content **after** the browser has parsed and normalized it. Most notably, some elements such as `<ul>`, `<ol>`, `<table>` and `<select>` have restrictions on what elements can appear inside them, and some elements such as `<option>` can only appear inside certain other elements.

This will lead to issues when using custom components with elements that have such restrictions, for example:

``` html
<table>
  <my-row>...</my-row>
</table>
```

The custom component `<my-row>` will be hoisted out as invalid content, thus causing errors in the eventual rendered output. A workaround is to use the `is` special attribute:

``` html
<table>
  <tr is="my-row"></tr>
</table>
```

**It should be noted that these limitations do not apply if you are using string templates from one of the following sources**:

- `<script type="text/x-template">`
- JavaScript inline template strings
- `.vue` components

Therefore, prefer using string templates whenever possible.

### `data` 는 반드시 Function 이어야 한다.

Most of the options that can be passed into the Vue constructor can be used in a component, with one special case: `data` must be function. In fact, if you try this:

``` js
Vue.component('my-component', {
  template: '<span>{{ message }}</span>',
  data: {
    message: 'hello'
  }
})
```

Then Vue will halt and emit warnings in the console, telling you that `data` must be a function for component instances. It's good to understand why the rules exist though, so let's cheat.

``` html
<div id="example-2">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
```

``` js
var data = { counter: 0 }

Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  // data is technically a function, so Vue won't
  // complain, but we return the same object
  // reference for each component instance
  data: function () {
    return data
  }
})

new Vue({
  el: '#example-2'
})
```

{% raw %}
<div id="example-2" class="demo">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
<script>
var data = { counter: 0 }
Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return data
  }
})
new Vue({
  el: '#example-2'
})
</script>
{% endraw %}

Since all three component instances share the same `data` object, incrementing one counter increments them all! Ouch. Let's fix this by instead returning a fresh data object:

``` js
data: function () {
  return {
    counter: 0
  }
}
```

Now all our counters each have their own internal state:

{% raw %}
<div id="example-2-5" class="demo">
  <my-component></my-component>
  <my-component></my-component>
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  }
})
new Vue({
  el: '#example-2-5'
})
</script>
{% endraw %}

### Composing Components

Components are meant to be used together, most commonly in parent-child relationships: component A may use component B in its own template. They inevitably need to communicate to one another: the parent may need to pass data down to the child, and the child may need to inform the parent of something that happened in the child. However, it is also very important to keep the parent and the child as decoupled as possible via a clearly-defined interface. This ensures each component's code can be written and reasoned about in relative isolation, thus making them more maintainable and potentially easier to reuse.

In Vue.js, the parent-child component relationship can be summarized as **props down, events up**. The parent passes data down to the child via **props**, and the child sends messages to the parent via **events**. Let's see how they work next.

<p style="text-align: center">
  <img style="width:300px" src="/images/props-events.png" alt="props down, events up">
</p>

## Props

### Props 이용한 데이터 전달

Every component instance has its own **isolated scope**. This means you cannot (and should not) directly reference parent data in a child component's template. Data can be passed down to child components using **props**.

A prop is a custom attribute for passing information from parent components. A child component needs to explicitly declare the props it expects to receive using the [`props` option](/api/#props):

``` js
Vue.component('child', {
  // declare the props
  props: ['message'],
  // just like data, the prop can be used inside templates
  // and is also made available in the vm as this.message
  template: '<span>{{ message }}</span>'
})
```

Then we can pass a plain string to it like so:

``` html
<child message="hello!"></child>
```

Result:

{% raw %}
<div id="prop-example-1" class="demo">
  <child message="hello!"></child>
</div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    child: {
      props: ['message'],
      template: '<span>{{ message }}</span>'
    }
  }
})
</script>
{% endraw %}

### camelCase 대 kebab-case

HTML attributes are case-insensitive, so when using non-string templates, camelCased prop names need to use their kebab-case (hyphen-delimited) equivalents:

``` js
Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```

``` html
<!-- kebab-case in HTML -->
<child my-message="hello!"></child>
```

Again, if you're using string templates, then this limitation does not apply.

### 동적인 Props

Similar to binding a normal attribute to an expression, we can also use `v-bind` for dynamically binding props to data on the parent. Whenever the data is updated in the parent, it will also flow down to the child:

``` html
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```

It's often simpler to use the shorthand syntax for `v-bind`:

``` html
<child :my-message="parentMsg"></child>
```

Result:

{% raw %}
<div id="demo-2" class="demo">
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    parentMsg: 'Message from parent'
  },
  components: {
    child: {
      props: ['myMessage'],
      template: '<span>{{myMessage}}</span>'
    }
  }
})
</script>
{% endraw %}

### Literal 대 Dynamic

A common mistake beginners tend to make is attempting to pass down a number using the literal syntax:

``` html
<!-- this passes down a plain string "1" -->
<comp some-prop="1"></comp>
```

However, since this is a literal prop, its value is passed down as a plain string `"1"` instead of an actual number. If we want to pass down an actual JavaScript number, we need to use `v-bind` so that its value is evaluated as a JavaScript expression:

``` html
<!-- this passes down an actual number -->
<comp v-bind:some-prop="1"></comp>
```

### 단방향의(One-Way) 데이터 흐름(flow)

All props form a **one-way-down** binding between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around. This prevents child components from accidentally mutating the parent's state, which can make your app's data flow harder to reason about.

In addition, every time the parent component is updated, all props in the child component will be refreshed with the latest value. This means you should **not** attempt to mutate a prop inside a child component. If you do, Vue will warn you in the console.

There are usually two cases where it's tempting to mutate a prop:

1. The prop is used to only pass in an initial value, the child component simply wants to use it as a local data property afterwards;

2. The prop is passed in as a raw value that needs to be transformed.

The proper answer to these use cases are:

1. Define a local data property that uses the prop's initial value as its initial value;

2. Define a computed property that is computed from the prop's value.

<p class="tip">Note that objects and arrays in JavaScript are passed by reference, so if the prop is an array or object, mutating the object or array itself inside the child **will** affect parent state.</p>

### Prop 유효성 검증

It is possible for a component to specify requirements for the props it is receiving. If a requirement is not met, Vue will emit warnings. This is especially useful when you are authoring a component that is intended to be used by others.

Instead of defining the props as an array of strings, you can use an object with validation requirements:

``` js
Vue.component('example', {
  props: {
    // basic type check (`null` means accept any type)
    propA: Number,
    // multiple possible types
    propB: [String, Number],
    // a required string
    propC: {
      type: String,
      required: true
    },
    // a number with default value
    propD: {
      type: Number,
      default: 100
    },
    // object/array defaults should be returned from a
    // factory function
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // custom validator function
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

The `type` can be one of the following native constructors:

- String
- Number
- Boolean
- Function
- Object
- Array

In addition, `type` can also be a custom constructor function and the assertion will be made with an `instanceof` check.

When a prop validation fails, Vue will produce a console warning (if using the development build).

## 확장 이벤트(Custom Events)

We have learned that the parent can pass data down to the child using props, but how do we communicate back to the parent when something happens? This is where Vue's custom event system comes in.

### 확장 이벤트에 `v-on` 사용하기

Every Vue instance implements an [events interface](/api/#Instance-Methods-Events), which means it can:

- Listen to an event using `$on(eventName)`
- Trigger an event using `$emit(eventName)`

<p class="tip">Note that Vue's event system is separate from the browser's [EventTarget API](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget). Though they work similarly, `$on` and `$emit` are __not__ aliases for `addEventListener` and `dispatchEvent`.</p>

In addition, a parent component can listen to the events emitted from a child component using `v-on` directly in the template where the child component is used.

Here's an example:

``` html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

``` js
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

{% raw %}
<div id="counter-event-example" class="demo">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
{% endraw %}

In this example, it's important to note that the child component is still completely decoupled from what happens outside of it. All it does is report information about its own activity, just in case a parent component might care.

#### Binding Native Events to Components

There may be times when you want to listen for a native event on the root element of a component. In these cases, you can use the `.native` modifier for `v-on`. For example:

``` html
<my-component v-on:click.native="doTheThing"></my-component>
```

### Form Input Components using Custom Events

This strategy can also be used to create custom form inputs that work with `v-model`. Remember:

``` html
<input v-model="something">
```

is just syntactic sugar for:

``` html
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

When used with a component, this simplifies to:

``` html
<input v-bind:value="something" v-on:input="something = arguments[0]">
```

So for a component to work with `v-model`, it must:

- accept a `value` prop
- emit an `input` event with the new value

Let's see it in action:

``` html
<div id="v-model-example">
  <p>{{ message }}</p>
  <my-input
    label="Message"
    v-model="message"
  ></my-input>
</div>
```

``` js
Vue.component('my-input', {
  template: '\
    <div class="form-group">\
      <label v-bind:for="randomId">{{ label }}:</label>\
      <input v-bind:id="randomId" v-bind:value="value" v-on:input="onInput">\
    </div>\
  ',
  props: ['value', 'label'],
  data: function () {
    return {
      randomId: 'input-' + Math.random()
    }
  },
  methods: {
    onInput: function (event) {
      this.$emit('input', event.target.value)
    }
  },
})

new Vue({
  el: '#v-model-example',
  data: {
    message: 'hello'
  }
})
```

{% raw %}
<div id="v-model-example" class="demo">
  <p>{{ message }}</p>
  <my-input
    label="Message"
    v-model="message"
  ></my-input>
</div>
<script>
Vue.component('my-input', {
  template: '\
    <div class="form-group">\
      <label v-bind:for="randomId">{{ label }}:</label>\
      <input v-bind:id="randomId" v-bind:value="value" v-on:input="onInput">\
    </div>\
  ',
  props: ['value', 'label'],
  data: function () {
    return {
      randomId: 'input-' + Math.random()
    }
  },
  methods: {
    onInput: function (event) {
      this.$emit('input', event.target.value)
    }
  },
})
new Vue({
  el: '#v-model-example',
  data: {
    message: 'hello'
  }
})
</script>
{% endraw %}

This interface can be used not only to connect with form inputs inside a component, but also to easily integrate input types that you invent yourself. Imagine these possibilities:

``` html
<voice-recognizer v-model="question"></voice-recognizer>
<webcam-gesture-reader v-model="gesture"></webcam-gesture-reader>
<webcam-retinal-scanner v-model="retinalImage"></webcam-retinal-scanner>
```

### Non Parent-Child Communication

Sometimes two components may need to communicate with one-another but they are not parent/child to each other. In simple scenarios, you can use an empty Vue instance as a central event bus:

``` js
var bus = new Vue()
```
``` js
// in component A's method
bus.$emit('id-selected', 1)
```
``` js
// in component B's created hook
bus.$on('id-selected', function (id) {
  // ...
})
```

In more complex cases, you should consider employing a dedicated [state-management pattern](/guide/state-management.html).

## Content Distribution with Slots

When using components, it is often desired to compose them like this:

``` html
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

There are two things to note here:

1. The `<app>` component does not know what content may be present inside its mount target. It is decided by whatever parent component that is using `<app>`.

2. The `<app>` component very likely has its own template.

To make the composition work, we need a way to interweave the parent "content" and the component's own template. This is a process called **content distribution** (or "transclusion" if you are familiar with Angular). Vue.js implements a content distribution API that is modeled after the current [Web Components spec draft](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md), using the special `<slot>` element to serve as distribution outlets for the original content.

### 컴파일 범위(scope)

API에 대해 더 깊이 들어가기 전에, 컴텐츠가 컴파일 되는 범위(scope)를 좀 더 명확하게 이해해 보자. 아래와 같은 템플릿이 있다고 해보자 :

``` html
<child-component>
  {{ message }}
</child-component>
```

 `message` 값은 parent의 `data`에 있어야 할까? 아니면 child-component 의 `data`에 있어야 할까? 정답은 `parent` 의 `data`에 있어야 한다이다. 컴포넌트 스콥(scope) 에 대해 우선적으로 적용되는 룰은 :

> parent 템플릿에서 컴파일되는 모든 것은 parent 스콥에 있어야 하고, child 템플릿 안에서 컴파일 되는 모든 것은 child 스콥에 있어야 한다. 

종종 나오는 실수가 바로 `child`의 속성이나 메쏘드를 `parent` 템플릿에서 호출(혹은 바인딩)하려고 하는 것이다. 

``` html
<!-- 이렇게 쓰면 동작하지 않는다. -->
<child-component v-show="someChildProperty"></child-component>
```

 `someChildProperty` 가 child 컴포넌트의 property 라고 하면, 위의 예제는 동작하지 않을 것이다. parent 템플릿은 child 컴포넌트가 어떤 메쏘드를 가지고 있고 어떤 `data`를 가지고 있는지 전혀 알지 못한다. 

 만약 child 스콥에 바인드 되는 directive를 root 노드의 template 에서 사용해야 한다면 child 컴포넌트의 template 으로 directive 들을 옮겨서 표현해주어야 한다 :

``` js
Vue.component('child-component', {
  // 해당 스콥에 someChildProperty 값이 존재하기 때문에 이 코드는 동작한다.
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```

마찬가지로, child에서 받아들이는 값도 parent 스콥에서 컴파일되어 전달된 내용이다.

### 단일 Slot

Parent content will be **discarded** unless the child component template contains at least one `<slot>` outlet. When there is only one slot with no attributes, the entire content fragment will be inserted at its position in the DOM, replacing the slot itself.

Anything originally inside the `<slot>` tags is considered **fallback content**. Fallback content is compiled in the child scope and will only be displayed if the hosting element is empty and has no content to be inserted.

Suppose we have a component called `my-component` with the following template:

``` html
<div>
  <h2>I'm the child title</h2>
  <slot>
    This will only be displayed if there is no content
    to be distributed.
  </slot>
</div>
```

And a parent that uses the component:

``` html
<div>
  <h1>I'm the parent title</h1>
  <my-component>
    <p>This is some original content</p>
    <p>This is some more original content</p>
  </my-component>
</div>
```

The rendered result will be:

``` html
<div>
  <h1>I'm the parent title</h1>
  <div>
    <h2>I'm the child title</h2>
    <p>This is some original content</p>
    <p>This is some more original content</p>
  </div>
</div>
```

### 이름을 준 Slots

`<slot>` elements have a special attribute, `name`, which can be used to further customize how content should be distributed. You can have multiple slots with different names. A named slot will match any element that has a corresponding `slot` attribute in the content fragment.

There can still be one unnamed slot, which is the **default slot** that serves as a catch-all outlet for any unmatched content. If there is no default slot, unmatched content will be discarded.

For example, suppose we have an `app-layout` component with the following template:

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

Parent markup:

``` html
<app-layout>
  <h1 slot="header">Here might be a page title</h1>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <p slot="footer">Here's some contact info</p>
</app-layout>
```

The rendered result will be:

``` html
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```

The content distribution API is a very useful mechanism when designing components that are meant to be composed together.

## 동적인 컴포넌트

You can use the same mount point and dynamically switch between multiple components using the reserved `<component>` element and dynamically bind to its `is` attribute:

``` js
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component v-bind:is="currentView">
  <!-- component changes when vm.currentView changes! -->
</component>
```

If you prefer, you can also bind directly to component objects:

``` js
var Home = {
  template: '<p>Welcome home!</p>'
}

var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

### `keep-alive`

If you want to keep the switched-out components in memory so that you can preserve their state or avoid re-rendering, you can wrap a dynamic component in a `<keep-alive>` element:

``` html
<keep-alive>
  <component :is="currentView">
    <!-- inactive components will be cached! -->
  </component>
</keep-alive>
```

Check out more details on `<keep-alive>` in the [API reference](/api/#keep-alive).

## 그밖에 

### 재사용 가능한 컴포넌트 만들기 

When authoring components, it's good to keep in mind whether you intend to reuse it somewhere else later. It's OK for one-off components to be tightly coupled, but reusable components should define a clean public interface and make no assumptions about the context it's used in.

The API for a Vue component comes in three parts - props, events, and slots:

- **Props** allow the external environment to pass data into the component

- **Events** allow the component to trigger side effects in the external environment

- **Slots** allow the external environment to compose the component with extra content.

With the dedicated shorthand syntaxes for `v-bind` and `v-on`, the intents can be clearly and succinctly conveyed in the template:

``` html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```

### 자식 컴포넌트를 참조할 수 있게 해주는 Refs

Despite the existence of props and events, sometimes you might still need to directly access a child component in JavaScript. To achieve this you have to assign a reference ID to the child component using `ref`. For example:

``` html
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// access child component instance
var child = parent.$refs.profile
```

When `ref` is used together with `v-for`, the ref you get will be an array or an object containing the child components mirroring the data source.

<p class="tip">`$refs` are only populated after the component has been rendered, and it is not reactive. It is only meant as an escape hatch for direct child manipulation - you should avoid using `$refs` in templates or computed properties.</p>

### 비동기화된 컴포넌트 (Async Components)

In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it's actually needed. To make that easier, Vue allows you to define your component as a factory function that asynchronously resolves your component definition. Vue will only trigger the factory function when the component actually needs to be rendered and will cache the result for future re-renders. For example:

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

The factory function receives a `resolve` callback, which should be called when you have retrieved your component definition from the server. You can also call `reject(reason)` to indicate the load has failed. The `setTimeout` here is simply for demonstration; How to retrieve the component is entirely up to you. One recommended approach is to use async components together with [Webpack's code-splitting feature](http://webpack.github.io/docs/code-splitting.html):

``` js
Vue.component('async-webpack-example', function (resolve) {
  // This special require syntax will instruct Webpack to
  // automatically split your built code into bundles which
  // are loaded over Ajax requests.
  require(['./my-async-component'], resolve)
})
```

You can also return a `Promise` in the resolve function, so with Webpack 2 + ES2015 syntax you can do:

``` js
Vue.component(
  'async-webpack-example',
  () => System.import('./my-async-component')
)
```

<p class="tip">If you're a <strong>Browserify</strong> user that would like to use async components, it's unfortunately not possible and probably never will be, as its creator has [made it clear](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224) that async loading "is not something that Browserify will ever support." If this is a feature that's important to you, we recommend using Webpack instead.</p>

### 컴포넌트 이름 생성 규칙 (Component Naming Conventions)

When registering components (or props), you can use kebab-case, camelCase, or TitleCase. Vue doesn't care.

``` js
// in a component definition
components: {
  // register using camelCase
  'kebab-cased-component': { /* ... */ },
  'camelCasedComponent': { /* ... */ },
  'TitleCasedComponent': { /* ... */ }
}
```

Within HTML templates though, you have to use the kebab-case equivalents:

``` html
<!-- alway use kebab-case in HTML templates -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<title-cased-component></title-cased-component>
```

When using _string_ templates however, we're not bound by HTML's case-insensitive restrictions. That means even in the template, you reference your components and props using camelCase, PascalCase, or kebab-case:

``` html
<!-- use whatever you want in string templates! -->
<my-component></my-component>
<myComponent></myComponent>
<MyComponent></MyComponent>
```

If your component isn't passed content via `slot` elements, you can even make it self-closing with a `/` after the name:

``` html
<my-component/>
```

Again, this _only_ works within string templates, as self-closing custom elements are not valid HTML and your browser's native parser will not understand them.

### Recursive Component

Components can recursively invoke themselves in their own template. However, they can only do so with the `name` option:

``` js
name: 'unique-name-of-my-component'
```

When you register a component globally using `Vue.component`, the global ID is automatically set as the component's `name` option.

``` js
Vue.component('unique-name-of-my-component', {
  // ...
})
```

If you're not careful, recursive components can also lead to infinite loops:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

A component like the above will result in a "max stack size exceeded" error, so make sure recursive invocation is conditional (i.e. uses a `v-if` that will eventually be `false`).

### 인라인 템플릿 (inline template)

템플릿의 child 컴포넌트에 특별히 `inline-template` 이라는 어트리뷰트를 줄 수 있는데, 이 경우 하위 엘리먼트가 child 컴포넌트의 template 을ㅇ 대체해서 템플릿으로 사용된다. 이렇게 함으로서 좀 더 유여한 템플릿 활용이 가능하다.

``` html
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>  
</my-component>
```

하지만, `inline-template` 을 사용하면 그 안에서 사용하는 변수의 스콥(scope)이 헷갈릴 수 있다. 따라서 급한 경우가 아니라면 가능한 child 컴포넌트의 `.vue` 파일에 `template` 엘리먼트로 표현하거나 child 생성시 옵션으로 `template`을 지정해서 넘기는 것이 훨씬 바람직하다. 

### X-Templates

템플릿을 만드는 또다른 방법으로, script 엘리먼트 아래 type을 `text/x-template` 으로 주고, id 값으로 레퍼런스 하는 방법이 있다. 예를 들어 :  

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

이런 방식은 아주 작은 애플리케이션이나 데모용으로 제작할때에 한해서 사용하는 것이 좋다. 이런 식으로 템플릿과 코드가 동떨어지게 작업하면 컴포넌트를 재사용하기가 어려워지고 큰 어플리케이션인 경우 소스 관리가 어려워지기 때문이다.

### `v-once` 를 이용해 가벼운 전역 컴포넌트 만들기

 단순 HTML 엘리먼트는 뷰에서 매우 빠르게 렌더링 되기 때문에 크게 이득은 없겠으나, 템플릿에 변수 없이 정해진 단순 HTML 로만 구성되는 경우가 있다. 이럴때는 `v-once` 라는 명령어(directive : 지시어)를 사용해서 템플릿이 한번만 컴파일 되도록 하고 이를 캐싱해 두었다가 매번 재사용하는 것이 훨씬 효일적이다. 

``` js
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Terms of Service</h1>\
      ... a lot of static content ...\
    </div>\
  '
})
```
