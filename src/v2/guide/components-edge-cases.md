---
title: Handling Edge Cases
type: guide
order: 106
---

> 이 페이지는 여러분이 이미 [컴포넌트 기초](components.html)를 읽었다고 가정하고 쓴 내용입니다. 컴포넌트가 처음이라면 기초 문서를 먼저 읽으시기 바랍니다.

<p class="tip"> 해당 페이지의 모든 기능들은 일반적이지 않은 상황을 위해 Vue의 기본 규칙에서 조금 벗어나게 되는 예외적인 상황에 대한 것입니다. 이러한 경우들에 대해 명백한 단점이 존재하며, 위험요소로 작용할 수 있음을 명심하세요. 각 상황에 대해 어떤 위험요소가 있는지가 작성되어 있으므로, 아래의 예외들을 적용하기 전에 꼭 위험요소를 확인하고 기억하세요.</p>

  

## 엘리먼트 & 컴포넌트 접근

대부분의 경우, 다른 컴포넌트에 접근하거나 직접 DOM 엘리먼트에 접근하는 것을 피하는 것이 좋습니다. 그럼에도 불구하고, 이러한 접근이 허용되는 경우가 있습니다. 

### 루트 엘리먼트에 접근하기

`new Vue` 의 모든 하위 컴포넌트에서는 `$root` 속성을 이용해 루트 인스턴스에 접근할 수 있습니다. 예를 들어, 아래와 같은 루트 인스턴스가 있다면:

```js
// 루트 Vue 인스턴스
new Vue({
  data: {
    foo: 1
  },
  computed: {
    bar: function () { /* ... */ }
  },
  methods: {
    baz: function () { /* ... */ }
  }
})
```

모든 하위 컴포넌트에서 아래와 같이 접근할 수 있으며, 전역 저장소처럼 활용할 수 있습니다.

```js
// root의 데이터 가져오기
this.$root.foo

// root의 데이터 수정하기
this.$root.foo = 2

// root의 computed 속성 접근하기
this.$root.bar

// root의 method 사용하기
this.$root.baz()
```

<p class="tip">이러한 패턴은 아주 작은 크기의 어플리케이션이나 적은 수의 컴포넌트에 대해서 유용하게 사용될수는 있습니다. 하지만 어플리케이션의 크기가 커지게 될 때 해당 패턴을 확장하기란 쉬운 일이 아닙니다. 대부분의 경우, 상태 관리를 위해 <a href="https://github.com/vuejs/vuex">Vuex</a> 를 사용하는 것을 강력히 권장합니다..</p>

### 부모 컴포넌트 인스턴스에 접근하기

`$root`와 비슷하게, `$parent` 속성을 사용하여 자식 요소에서 부모 인스턴스에 접근할 수 있습니다. 이는 prop을 이용해 데이터를 넘겨주는 것의 (조금 뒤떨어지는) 대안으로써 사용할 수 있습니다.

<p class="tip"> 대부분의 경우, 특히 부모 요소의 데이터를 자식 요소에서 변경하는 경우에 부모 요소에 접근하는 것은 디버깅의 편의성과 코드 가독성을 크게 해칩니다. 나중에 해당 컴포넌트를 다시 보았을 때, 어디서 변경이 발생하였는지를 추적하는 것이 굉장히 어려워 질 수 있습니다.</p>

하지만 가끔, 부분적으로 컴포넌트간의 공유가 *이루어져야 하는* 라이브러리가 존재합니다. 예를 들어, HTML을 렌더링하는 대신 JavaScript API와 통신하는  Google Maps 컴포넌트 같은 경우를 예로 들 수 있습니다:

```html
<google-map>
  <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
</google-map>
```

이 `<google-map>` 컴포넌트는 모든 하위 컴포넌트가 접근할 수 있어야 하는 `map` 속성을 가져야 합니다. 위의 경우, `<google-map-marker>`가 `this.$parent.getMap`과 같은 방식으로 map에 접근할 수 있어야 정상적으로 마커를 추가할 수 있을 것입니다. [여기](https://jsfiddle.net/chrisvfritz/ttzutdxh/) 에서 해당 패턴을 좀 더 자세히 확인할 수 있습니다. 

본질적으로 위와 같은 패턴은 여전히 취약하다는 것을 기억하세요. 예를 들어, `<google-map-region>` 컴포넌트를 추가하고 `<google-map-markers>` 컴포넌트가 그 지역 안에서만 마커를 렌더링 할 수도 있도록 구조를 변경한다고 가정해 봅시다.

```html
<google-map>
  <google-map-region v-bind:shape="cityBoundaries">
    <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
  </google-map-region>
</google-map>
```

이 경우, `<google-map-markers>` 안에 아래와 같은 코드를 쓰게 될 것입니다.

```js
var map = this.$parent.map || this.$parent.$parent.map
```

이러한 패턴은 금세 더 이상 손댈 수 없게 변하게 됩니다. 이러한 이유로, 임의의 깊이를 가진 하위 요소에게 컨텍스트 정보를 제공하기 위한 방법으로써 위 패턴 대신에 [의존성 주입](#의존성-주입)을 사용하는 것이 권장됩니다.

### 자식 컴포넌트의 인스턴스 및 요소에 접근하기

물론 props와 events가 존재하지만, 가끔 JavaScript에서 자식 요소에 직접 접근해야 하는 경우가 있습니다. 이 경우, `ref` 속성을 이용해 자식 요소에 레퍼런스 ID를 할당하여 해결할 수 있습니다. 예를 들어:

```html
<base-input ref="usernameInput"></base-input>
```

이제 `ref`를 이용해 ID를 정의한 컴포넌트 안에서 아래와 같이 작성하면:

```js
this.$refs.usernameInput
```

 `<base-input>` 인스턴스에 접근할 수 있습니다. 이러한 방식은 필요에 따라서 유용할 수 있습니다. 예를 들어, 프로그래밍적으로 부모요소에서 자식 요소 내부의 input 요소에 focus를 부여하고 싶은 경우를 생각해 봅시다. 이 경우 `<base-input>` 컴포넌트에서도 `ref`를 사용함으로써 특정 요소에 접근할 수 있습니다. 예를 들어 `<base-input>` 내에

```html
<input ref="input">
```

와 같이 작성되어 있고 부모요소에서 아래와 같은 메소드를 작성함으로써:

```js
methods: {
  // Used to focus the input from the parent
  focus: function () {
    this.$refs.input.focus()
  }
}
```

부모요소에서 `<base-input>` 내부의 input에 focus되도록 하기 위해 아래와 같이 작성할 수 있습니다.

```js
this.$refs.usernameInput.focus()
```

`ref` 를 `v-for` 와 함께 사용하는 경우에는 ref는 데이터 소스를 미러링하는 자식 컴포넌트를 포함하는 배열을 얻게 됩니다.

<p class="tip"><code>$refs</code> 는 컴포넌트가 렌더링 된 후에 접근할 수 있으며, 반응형이 아닙니다. 즉, 직접적인 자식 요소 제어에만 유효합니다 - <code>$refs</code>를 template나 computed 속성 안에 포함시키지 않는 것이 좋습니다.</p>

### 의존성 주입

이전에 [부모 컴포넌트 인스턴스에 접근하기](#부모-컴포넌트-인스턴스에-접근하기)에서 아래와 같은 예제를 다루었습니다.

```html
<google-map>
  <google-map-region v-bind:shape="cityBoundaries">
    <google-map-markers v-bind:places="iceCreamShops"></google-map-markers>
  </google-map-region>
</google-map>
```

이 컴포넌트에서, `<google-map>`의 모든 하위 자식들은 어떤 지도와 상호작용해야 하는지를 알기 위해  `getMap` 메소드에 접근할 수 있었어야 했습니다. 불행히도 `$parent` 속성은 여러 번 중첩된 컴포넌트에 대해서 확장 가능한 방법이 아니었습니다. 이 경우, `provide` 와 `inject` 두 개의 옵션을 사용하는 의존성 주입을 유용하게 사용할 수 있습니다.

`provide` 옵션은 모든 하위 자식들에게 **제공하고자 하는** data 및 methods를 특정할 수 있게 해 줍니다. 이 경우에는 `<google-map>` 안의 `getMap` 메소드가 제공하고자 하는 메소드입니다.

```js
provide: function () {
  return {
    getMap: this.getMap
  }
}
```

이제 모든 하위 자식들에서 `inject` 옵션을 이용해 특정 속성을 받아 추가할 수 있도록 할 수 있습니다:

```js
inject: ['getMap']
```

전체 예제 코드는 [여기](https://jsfiddle.net/chrisvfritz/tdv8dt3s/)에서 볼 수 있습니다. `$parent` 를 사용하는 것에 비해서 얻을 수 있는 이점은 이제 모든 하위 자식에서 `<google-map>` 의 모든 요소에 접근하지 않고도  `getMap` 메소드에 접근할 수 있다는 것입니다. 이는 컴포넌트를 개발할 때 자식 요소가 의존성을 갖는 바꾸거나 제거할 수도 있다는 두려움 없이 안전하게 개발할 수 있도록 해 줍니다. 컴포넌트들 사이의 인터페이스는 `props` 로써 명료하게 정의되어 있게 됩니다.

사실 의존성 주입은 아래의 것들을 제외하면 "장거리 props"라고 생각할 수 있습니다:

* 조상 컴포넌트는 어떤 자손 컴포넌트가 제공한 속성을 사용했는지 알 필요가 없습니다.
* 자손 컴포넌트는 inject된 속성이 어디에서 왔는지 알 필요가 없습니다.

<p class="tip"> 의존성 주입에도 안좋은 면이 있습니다. 의존성 주입은 어플리케이션 안의 컴포넌트들을 현재의 구조로 묶으며, 리팩토링을 어렵게 만듭니다. 전달된 속성들 또한 반응형이 아닙니다. 이는 디자인적으로 의존성 주입을 중앙 집중형 데이터 저장소로 사용하는 것이 <a href="#루트-엘리먼트에-접근하기">루트 엘리먼트에 접근하기</a>에 언급된 것과 같은 이유로 확장에 용이하지 않다는 뜻입니다. 공유하고자 하는 속성이 일반적이지 않고 앱에 특정되어 있거나 조상 요소로부터 제공된 데이터를 수정할 필요가 있다면 <a href="https://github.com/vuejs/vuex">Vuex</a>와 같은 실제 상태 관리 솔루션이 필요하다는 좋은 신호입니다.</p>

의존성 주입에 대해서 [API 문서](https://vuejs.org/v2/api/#provide-inject) 에서 더 알아보세요.

## Programmatic Event Listeners

So far, you've seen uses of `$emit`, listened to with `v-on`, but Vue instances also offer other methods in its events interface. We can:

- Listen for an event with `$on(eventName, eventHandler)`
- Listen for an event only once with `$once(eventName, eventHandler)`
- Stop listening for an event with `$off(eventName, eventHandler)`

You normally won't have to use these, but they're available for cases when you need to manually listen for events on a component instance. They can also be useful as a code organization tool. For example, you may often see this pattern for integrating a 3rd-party library:

```js
// Attach the datepicker to an input once
// it's mounted to the DOM.
mounted: function () {
  // Pikaday is a 3rd-party datepicker library
  this.picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
  })
},
// Right before the component is destroyed,
// also destroy the datepicker.
beforeDestroy: function () {
  this.picker.destroy()
}
```

This has two potential issues:

- It requires saving the `picker` to the component instance, when it's possible that only lifecycle hooks need access to it. This isn't terrible, but it could be considered clutter.
- Our setup code is kept separate from our cleanup code, making it more difficult to programmatically clean up anything we set up.

You could resolve both issues with a programmatic listener:

```js
mounted: function () {
  var picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
  })

  this.$once('hook:beforeDestroy', function () {
    picker.destroy()
  })
}
```

Using this strategy, we could even use Pikaday with several input elements, with each new instance automatically cleaning up after itself:

```js
mounted: function () {
  this.attachDatepicker('startDateInput')
  this.attachDatepicker('endDateInput')
},
methods: {
  attachDatepicker: function (refName) {
    var picker = new Pikaday({
      field: this.$refs[refName],
      format: 'YYYY-MM-DD'
    })

    this.$once('hook:beforeDestroy', function () {
      picker.destroy()
    })
  }
}
```

See [this fiddle](https://jsfiddle.net/09jvu65m/) for the full code. Note, however, that if you find yourself having to do a lot of setup and cleanup within a single component, the best solution will usually be to create more modular components. In this case, we'd recommend creating a reusable `<input-datepicker>` component.

To learn more about programmatic listeners, check out the API for [Events Instance Methods](https://vuejs.org/v2/api/#Instance-Methods-Events).

<p class="tip">Note that Vue's event system is different from the browser's <a href="https://developer.mozilla.org/en-US/docs/Web/API/EventTarget">EventTarget API</a>. Though they work similarly, <code>$emit</code>, <code>$on</code>, and <code>$off</code> are <strong>not</strong> aliases for <code>dispatchEvent</code>, <code>addEventListener</code>, and <code>removeEventListener</code>.</p>

## Circular References

### Recursive Components

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

### Circular References Between Components

Let's say you're building a file directory tree, like in Finder or File Explorer. You might have a `tree-folder` component with this template:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

Then a `tree-folder-contents` component with this template:

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

When you look closely, you'll see that these components will actually be each other's descendent _and_ ancestor in the render tree - a paradox! When registering components globally with `Vue.component`, this paradox is resolved for you automatically. If that's you, you can stop reading here.

However, if you're requiring/importing components using a __module system__, e.g. via Webpack or Browserify, you'll get an error:

```
Failed to mount component: template or render function not defined.
```

To explain what's happening, let's call our components A and B. The module system sees that it needs A, but first A needs B, but B needs A, but A needs B, etc. It's stuck in a loop, not knowing how to fully resolve either component without first resolving the other. To fix this, we need to give the module system a point at which it can say, "A needs B _eventually_, but there's no need to resolve B first."

In our case, let's make that point the `tree-folder` component. We know the child that creates the paradox is the `tree-folder-contents` component, so we'll wait until the `beforeCreate` lifecycle hook to register it:

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
}
```

Or alternatively, you could use Webpack's asynchronous `import` when you register the component locally:

``` js
components: {
  TreeFolderContents: () => import('./tree-folder-contents.vue')
}
```

Problem solved!

## Alternate Template Definitions

### Inline Templates

When the `inline-template` special attribute is present on a child component, the component will use its inner content as its template, rather than treating it as distributed content. This allows more flexible template-authoring.

``` html
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>
</my-component>
```

Your inline template needs to be defined inside the DOM element to which Vue is attached.

<p class="tip">However, <code>inline-template</code> makes the scope of your templates harder to reason about. As a best practice, prefer defining templates inside the component using the <code>template</code> option or in a <code>&lt;template&gt;</code> element in a <code>.vue</code> file.</p>

### X-Templates

Another way to define templates is inside of a script element with the type `text/x-template`, then referencing the template by an id. For example:

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

Your x-template needs to be defined outside the DOM element to which Vue is attached.

<p class="tip">These can be useful for demos with large templates or in extremely small applications, but should otherwise be avoided, because they separate templates from the rest of the component definition.</p>

## Controlling Updates

Thanks to Vue's Reactivity system, it always knows when to update (if you use it correctly). There are edge cases, however, when you might want to force an update, despite the fact that no reactive data has changed. Then there are other cases when you might want to prevent unnecessary updates.

### Forcing an Update

<p class="tip">If you find yourself needing to force an update in Vue, in 99.99% of cases, you've made a mistake somewhere.</p>

You may not have accounted for change detection caveats [with arrays](https://vuejs.org/v2/guide/list.html#Caveats) or [objects](https://vuejs.org/v2/guide/list.html#Object-Change-Detection-Caveats), or you may be relying on state that isn't tracked by Vue's reactivity system, e.g. with `data`.

However, if you've ruled out the above and find yourself in this extremely rare situation of having to manually force an update, you can do so with [`$forceUpdate`](../api/#vm-forceUpdate).

### Cheap Static Components with `v-once`

Rendering plain HTML elements is very fast in Vue, but sometimes you might have a component that contains **a lot** of static content. In these cases, you can ensure that it's only evaluated once and then cached by adding the `v-once` directive to the root element, like this:

``` js
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... a lot of static content ...
    </div>
  `
})
```

<p class="tip">Once again, try not to overuse this pattern. While convenient in those rare cases when you have to render a lot of static content, it's simply not necessary unless you actually notice slow rendering -- plus, it could cause a lot of confusion later. For example, imagine another developer who's not familiar with <code>v-once</code> or simply misses it in the template. They might spend hours trying to figure out why the template isn't updating correctly.</p>
