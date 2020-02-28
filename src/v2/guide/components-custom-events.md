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

`v-model` 을 사용하는 컴포넌트는 기본값으로 `value`를 prop으로 사용하고, `input` 을 이벤트로 사용합니다. 이 때, 체크박스와 같이 `value` 속성을  [다른 용도](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#Value)로 사용하여야 하는 경우에는 `model` 옵션을 이용하여 문제가 생기는 것을 방지할 수 있습니다.

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

이제  `v-model` 을 컴포넌트에 사용하게 되면:

```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```

`lovingVue` 의 값은 `checked` prop으로 전달됩니다. 그리고 `lovingVue` 속성은 `<base-checkbox>` 가 `change` 이벤트를 emit할 때 새로운 값으로 업데이트됩니다. 

이 때, `checked` 속성을 컴포넌트의 `props` 옵션에 선언해 주어야 하는 것을 잊지 마세요.

## 네이티브 이벤트를 컴포넌트에 바인딩 하기

컴포넌트에서 루트 엘리먼트의 네이티브 이벤트를 직접 감지하고 싶은 경우가 있을 수 있습니다. 이 경우, `v-on`에 `.native` 수식어를 사용할 수 있습니다. 

```html
<base-input v-on:focus.native="onFocus"></base-input>
```

위와 같은 테크닉은 경우에 따라 유용하긴 하지만 `<input>` 과 같이 특수한 경우엔 좋지 않은 선택일 수 있습니다. 예를 들어, `<base-input>` 이라는 컴포넌트의 루트 엘리먼트가 실제로는 `<label>` 엘리먼트인 경우를 생각해 볼 수 있습니다.

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

이 경우  `.native` 수식어를 사용해 바인딩된 부모 엘리먼트의 리스너는 별다른 에러를 발생시키지 않으면서 `onFocus` 핸들러는 의도한 대로 동작하지 않을 것입니다.

이러한 문제를 해결하기 위해서 Vue는 컴포넌트에서 사용된 리스너를 포함하는 오브젝트인  `$listeners` 속성을 제공합니다. 예를 들어:

```js
{
  focus: function (event) { /* ... */ }
  input: function (value) { /* ... */ },
}
```

 `$listeners` 속성을 이용하면 컴포넌트에서 `v-on=$listeners`를 이용해 부모 엘리먼트가 가진 이벤트 리스너를 특정 자식 엘리먼트에게 전달할 수 있습니다.  가령 `<input>`같은 엘리먼트에 `v-model` 를 적용하고 싶은 경우라면, 아래와 같이 `inputListeners`같은 새로운 computed 속성을 생성하여 유용하게 활용할 수 있습니다. 

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` 는 오브젝트를 새로운 오브젝트로 병합합니다.
      return Object.assign({},
        // 우선 부모 엘리먼트의 모든 리스너를 추가합니다.
        this.$listeners,
        // 그 다음, 기존 리스너를 override하는
        // 커스텀 리스터를 추가할 수 있습니다. 
        {
          // 아래 구문을 사용하면 v-model과 같이 동작하도록 만들 수 있습니다.
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

이제 `<base-input>` 컴포넌트는 **완전히 투명한** --즉, 일반적인 `<input>`과 완전히 동일하게 동작하는-- wrapper라고 볼 수 있습니다. 모든 속성과 리스너가 `.native` 수식어 없이도 기존과 동일하게 동작합니다.

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
