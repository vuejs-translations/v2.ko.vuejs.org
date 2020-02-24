---
title: 커스텀 이벤트
type: guide
order: 103
---

> 이 페이지는 여러분이 이미 [컴포넌트 기초](components.html)를 읽었다고 가정하고 쓴 내용입니다. 컴포넌트가 처음이라면 기초 문서를 먼저 읽으시기 바랍니다.

## 이벤트 이름

컴포넌트 및 props와는 달리, 이벤트는 자동 대소문자 변환을 제공하지 않습니다. 대소문자를 혼용하는 대신에 emit할 정확한 이벤트 이름을 작성하는 것이 권장됩니다. 예를 들어, 아래와 같이 camelCase로 작성된 이벤트를 emit하는 경우,

```js
this.$emit('myEvent')
```

아래와 같이 kebab-case로 이벤트를 작성하게 되면 아무 동작도 하지 않습니다.

```html
<!-- 이벤트가 동작하지 않음 -->
<my-component v-on:my-event="doSomething"></my-component>
```

컴포넌트 및 props와는 다르게 이벤트 이름은 자바스크립트 변수나 속성의 이름으로 사용되는 경우가 없으며, 따라서 camelCase나 PascalCase를 사용할 필요가 없습니다. 또한, (HTML의 대소문자 구분을 위해) DOM 템플릿의 `v-on` 이벤트리스너는 항상 자동으로 소문자 변환되기 때문에 `v-on:myEvent` 는 자동으로 `v-on:myevent`로 변환됩니다. -- 즉, `myEvent` 이벤트를 들을 수 없습니다.

이러한 이유 때문에, **이벤트 이름에는 kebab-case를 사용**하는것이 권장됩니다.

## 컴포넌트의 `v-model` 커스터마이징

> New in 2.2.0+

By default, `v-model` on a component uses `value` as the prop and `input` as the event, but some input types such as checkboxes and radio buttons may want to use the `value` attribute for a [different purpose](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#Value). Using the `model` option can avoid a conflict in such cases:

```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```

Now when using `v-model` on this component:

```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```

the value of `lovingVue` will be passed to the `checked` prop. The `lovingVue` property will then be updated when `<base-checkbox>` emits a `change` event with a new value.

<p class="tip">Note that you still have to declare the <code>checked</code> prop in the component's <code>props</code> option.</p>

## 네이티브 이벤트를 컴포넌트에 바인딩 하기

There may be times when you want to listen directly to a native event on the root element of a component. In these cases, you can use the `.native` modifier for `v-on`:

```html
<base-input v-on:focus.native="onFocus"></base-input>
```

This can be useful sometimes, but it's not a good idea when you're trying to listen on a very specific element, like an `<input>`. For example, the `<base-input>` component above might refactor so that the root element is actually a `<label>` element:

```html
<label>
  {{ label }}
  <input
    v-bind="$attrs"
    v-bind:value="value"
    v-on:input="$emit('input', $event.target.value)"
  >
</label>
```

In that case, the `.native` listener in the parent would silently break. There would be no errors, but the `onFocus` handler wouldn't be called when we expected it to.

To solve this problem, Vue provides a `$listeners` property containing an object of listeners being used on the component. For example:

```js
{
  focus: function (event) { /* ... */ }
  input: function (value) { /* ... */ },
}
```

Using the `$listeners` property, you can forward all event listeners on the component to a specific child element with `v-on="$listeners"`. For elements like `<input>`, that you also want to work with `v-model`, it's often useful to create a new computed property for listeners, like `inputListeners` below:

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` merges objects together to form a new object
      return Object.assign({},
        // We add all the listeners from the parent
        this.$listeners,
        // Then we can add custom listeners or override the
        // behavior of some listeners.
        {
          // This ensures that the component works with v-model
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```

Now the `<base-input>` component is a **fully transparent wrapper**, meaning it can be used exactly like a normal `<input>` element: all the same attributes and listeners will work, without the `.native` modifier.

## `.sync` 수식어

> New in 2.3.0+

In some cases, we may need "two-way binding" for a prop. Unfortunately, true two-way binding can create maintenance issues, because child components can mutate the parent without the source of that mutation being obvious in both the parent and the child.

That's why instead, we recommend emitting events in the pattern of `update:myPropName`. For example, in a hypothetical component with a `title` prop, we could communicate the intent of assigning a new value with:

```js
this.$emit('update:title', newTitle)
```

Then the parent can listen to that event and update a local data property, if it wants to. For example:

```html
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```

For convenience, we offer a shorthand for this pattern with the `.sync` modifier:

```html
<text-document v-bind:title.sync="doc.title"></text-document>
```

<p class="tip">Note that <code>v-bind</code> with the <code>.sync</code> modifier does <strong>not</strong> work with expressions (e.g. <code>v-bind:title.sync="doc.title + '!'"</code> is invalid). Instead, you must only provide the name of the property you want to bind, similar to <code>v-model</code>.</p>

The `.sync` modifier can also be used with `v-bind` when using an object to set multiple props at once:

```html
<text-document v-bind.sync="doc"></text-document>
```

This passes each property in the `doc` object (e.g. `title`) as an individual prop, then adds `v-on` update listeners for each one.

<p class="tip">Using <code>v-bind.sync</code> with a literal object, such as in <code>v-bind.sync="{ title: doc.title }"</code>, will not work, because there are too many edge cases to consider in parsing a complex expression like this.</p>
