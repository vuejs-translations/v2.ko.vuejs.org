---
title: Props
type: guide
order: 102
---

> 이 페이지는 여러분이 이미 [컴포넌트 기초](components.html)를 읽었다고 가정하고 쓴 내용입니다. 컴포넌트가 처음이라면 기초 문서를 먼저 읽으시기 바랍니다.

## Prop 대소문자 구분 (camelCase vs kebab-case)

HTML 어트리뷰트는 대소문자 구분이 없기 때문에 브라우저는 대문자를 소문자로 변경하여 읽습니다. 그렇기 때문에 카멜 케이스(대소문자 혼용)로 prop의 이름을 정한 경우에 DOM 템플릿 안에서는 케밥 케이스(하이픈으로-연결된-구조)를 사용하여야 올바르게 동작합니다.

``` js
Vue.component('blog-post', {
  // JavaScript에서의 camelCase
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

``` html
<!-- HTML에서의 kebab-case -->
<blog-post post-title="hello!"></blog-post>
```

물론, 문자열 템플릿을 사용하는 경우에는 무관합니다.

## Prop 타입

이때까지 우리는 문자열 배열 형태로 작성된 prop의 리스트를 보아 왔습니다.

```js
props: ['title', 'likes', 'isPublished', 'commentIds', 'author']
```

일반적으로 생각해 보면, prop에 특정 타입의 값을 넣고싶은 경우가 있을 수 있습니다. 이 때, 다음과 같이 prop을 속성 이름과 타입을 포함하는 오브젝트로 선언함으로써 타입이 지정된 prop의 리스트를 구현할 수 있습니다.

```js
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```

이는 컴포넌트를 읽기 좋게 문서화할 뿐 아니라 브라우저의 자바스크립트 콘솔에서도 잘못된 타입이 전달된 경우 경고를 띄워줄 수 있도록 해줍니다. 페이지 아래에 있는 [타입 체크와 prop 유효성 검사](#Prop-Validation) 에서 조금 더 자세한 내용을 확인할 수 있습니다.

## 정적 & 동적 prop 전달하기

아래와 같이 정적인 prop을 전달할 수 있습니다. 

```html
<blog-post title="My journey with Vue"></blog-post>
```

혹은 아래와 같이 `v-bind` 를 이용하여 동적인 prop을 전달할 수도 있습니다. 

```html
<!-- 변수에 담긴 값을 동적으로 할당 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 복잡한 표현식의 값을 동적으로 할당 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```

위 두 가지 경우는 모두 문자열 형태의 변수를 전달하지만 실제로는 *모든* 타입의 변수가 prop으로 전달될 수 있습니다.

### 숫자형(Number) 전달

```html
<!-- `42`는 정적인 값이지만, Vue에서 해당 값이 숫자라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줍니다.      -->
<blog-post v-bind:likes="42"></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있습니다. -->
<blog-post v-bind:likes="post.likes"></blog-post>
```

### 논리 자료형(Boolean) 전달

```html
<!-- 값이 없는 prop은 `true` 를 전달합니다.. -->
<blog-post is-published></blog-post>

<!-- `false`는 정적인 값이지만, Vue에서 해당 값이 논리 자료형이라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줍니다.                -->
<blog-post v-bind:is-published="false"></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있습니다. -->
<blog-post v-bind:is-published="post.isPublished"></blog-post>
```

### 배열(Array) 전달

```html
<!-- 해당 배열은 정적인 값이지만, Vue에서 해당 값이 배열이라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줍니다.          -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있습니다. -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

### 오브젝트(Object) 전달

```html
<!-- 해당 오브젝트는 정적인 값이지만, Vue에서 해당 값이 배열이라는 것을 알 수 있도록 하기 위해 -->
<!-- v-bind를 이용해 문자열이 아닌 JavaScript 표현식이라는 것을 알려줍니다.             -->
<blog-post
  v-bind:author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>

<!-- 변수의 값을 동적으로 할당할 수도 있습니다. -->
<blog-post v-bind:author="post.author"></blog-post>
```

### 오브젝트의 속성(Properties) 전달

오브젝트의 모든 속성을 전달하길 원하는 경우, `v-bind:prop-name` 대신 `v-bind` 만 작성함으로써 모든 속성을 prop으로 전달할 수 있습니다. 예를 들어, 아래와 같은 `post` 라는 오브젝트가 있다면:

``` js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

아래와 같이 작성하는 것은

``` html
<blog-post v-bind="post"></blog-post>
```

아래와 같이 동작합니다. 

``` html
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

## One-Way Data Flow

All props form a **one-way-down binding** between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around. This prevents child components from accidentally mutating the parent's state, which can make your app's data flow harder to understand.

In addition, every time the parent component is updated, all props in the child component will be refreshed with the latest value. This means you should **not** attempt to mutate a prop inside a child component. If you do, Vue will warn you in the console.

There are usually two cases where it's tempting to mutate a prop:

1. **The prop is used to pass in an initial value; the child component wants to use it as a local data property afterwards.** In this case, it's best to define a local data property that uses the prop as its initial value:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
  ```

2. **The prop is passed in as a raw value that needs to be transformed.** In this case, it's best to define a computed property using the prop's value:

  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Note that objects and arrays in JavaScript are passed by reference, so if the prop is an array or object, mutating the object or array itself inside the child component **will** affect parent state.</p>

## Prop Validation

Components can specify requirements for their props, such as the types you've already seen. If a requirement isn't met, Vue will warn you in the browser's JavaScript console. This is especially useful when developing a component that's intended to be used by others.

To specify prop validations, you can provide an object with validation requirements to the value of `props`, instead of an array of strings. For example:

``` js
Vue.component('my-component', {
  props: {
    // Basic type check (`null` and `undefined` values will pass any type validation)
    propA: Number,
    // Multiple possible types
    propB: [String, Number],
    // Required string
    propC: {
      type: String,
      required: true
    },
    // Number with a default value
    propD: {
      type: Number,
      default: 100
    },
    // Object with a default value
    propE: {
      type: Object,
      // Object or array defaults must be returned from
      // a factory function
      default: function () {
        return { message: 'hello' }
      }
    },
    // Custom validator function
    propF: {
      validator: function (value) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

When prop validation fails, Vue will produce a console warning (if using the development build).

<p class="tip">Note that props are validated **before** a component instance is created, so instance properties (e.g. `data`, `computed`, etc) will not be available inside `default` or `validator` functions.</p>

### Type Checks

The `type` can be one of the following native constructors:

- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol

In addition, `type` can also be a custom constructor function and the assertion will be made with an `instanceof` check. For example, given the following constructor function exists:

```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```

You could use:

```js
Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```

to validate that the value of the `author` prop was created with `new Person`.

## Non-Prop Attributes

A non-prop attribute is an attribute that is passed to a component, but does not have a corresponding prop defined.

While explicitly defined props are preferred for passing information to a child component, authors of component libraries can't always foresee the contexts in which their components might be used. That's why components can accept arbitrary attributes, which are added to the component's root element.

For example, imagine we're using a 3rd-party `bootstrap-date-input` component with a Bootstrap plugin that requires a `data-date-picker` attribute on the `input`. We can add this attribute to our component instance:

``` html
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```

And the `data-date-picker="activated"` attribute will automatically be added to the root element of `bootstrap-date-input`.

### Replacing/Merging with Existing Attributes

Imagine this is the template for `bootstrap-date-input`:

``` html
<input type="date" class="form-control">
```

To specify a theme for our date picker plugin, we might need to add a specific class, like this:

``` html
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```

In this case, two different values for `class` are defined:

- `form-control`, which is set by the component in its template
- `date-picker-theme-dark`, which is passed to the component by its parent

For most attributes, the value provided to the component will replace the value set by the component. So for example, passing `type="text"` will replace `type="date"` and probably break it! Fortunately, the `class` and `style` attributes are a little smarter, so both values are merged, making the final value: `form-control date-picker-theme-dark`.

### Disabling Attribute Inheritance

If you do **not** want the root element of a component to inherit attributes, you can set `inheritAttrs: false` in the component's options. For example:

```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```

This can be especially useful in combination with the `$attrs` instance property, which contains the attribute names and values passed to a component, such as:

```js
{
  required: true,
  placeholder: 'Enter your username'
}
```

With `inheritAttrs: false` and `$attrs`, you can manually decide which element you want to forward attributes to, which is often desirable for [base components](../style-guide/#Base-component-names-strongly-recommended):

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```

<p class="tip">Note that `inheritAttrs: false` option does **not** affect `style` and `class` bindings.</p>

This pattern allows you to use base components more like raw HTML elements, without having to care about which element is actually at its root:

```html
<base-input
  v-model="username"
  required
  placeholder="Enter your username"
></base-input>
```
