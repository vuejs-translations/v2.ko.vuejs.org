---
title: 예외적인 상황들
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

이 `<google-map>` 컴포넌트는 모든 하위 컴포넌트가 접근할 수 있어야 하는 `map` 속성을 가져야 합니다. 위의 경우, `<google-map-marker>`가 `this.$parent.getMap`과 같은 방식으로 `map`에 접근할 수 있어야 정상적으로 마커를 추가할 수 있을 것입니다. [여기](https://jsfiddle.net/chrisvfritz/ttzutdxh/) 에서 해당 패턴을 좀 더 자세히 확인할 수 있습니다. 

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

## 프로그래밍적 이벤트 리스너

지금까지 본 `$emit`을 사용하고 `v-on` 으로 듣는 방법 외에도 Vue 인스턴스는 또다른 이벤트 인터페이스 사용 방법을 가지고 있습니다. 우리는 다음과 같이 작성할 수 있습니다:

- `$on(eventName, eventHandler)` 을 이용한 이벤트 청취
- `$once(eventName, eventHandler)` 를 이용한 단발성 이벤트 청취
- `$off(eventName, eventHandler)` 를 이용한 이벤트 청취 중단

위의 방법들은 일반적으로 사용할 일이 없지만 컴포넌트 인스턴스 안의 이벤트를 수동적으로 청취하고자 하는 경우에 사용할 수 있습니다. 또한 코드를 조직화하는 도구로써 유용하게 사용될 수 있습니다. 예를 들어, 아래와 같이 서드파티 라이브러리를 사용하는 경우가 있습니다:

```js
// datepicker를 input에 한 번 연결합니다.
// DOM에 직접 연결됩니다.
mounted: function () {
  // Pikaday는 서드파티 라이브러리입니다.
  this.picker = new Pikaday({
    field: this.$refs.input,
    format: 'YYYY-MM-DD'
  })
},
// 컴포넌트를 destroy하기 직전에
// datepicker를 destroy합니다.
beforeDestroy: function () {
  this.picker.destroy()
}
```

이는 두 가지의 잠재적 이슈를 갖습니다:

- 라이프사이클 훅에서만 `picker`에 접근할 수 있는 경우, `picker` 가 컴포넌트 인스턴스 안에 저장되어야 합니다. 끔찍한 정도는 아니지만, 다소 어색하게 느껴질 수 있습니다.
- 셋업을 위한 코드와 제거를 위한 코드가 분리되어 있기에 무언가를 제거하거나 설치하는데 있어 (프로그래밍적으로) 어려워집니다.

프로그래밍적 리스너를 이용하면 위 두 가지 이슈를 모두 해결할 수 있습니다:

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

이러한 방법을 이용하면 우리는 Pikaday를 가양한 input 엘리먼트에 사용할 수 있으며, 각각의 새로운 인스턴스는 사용되고 난 후 자동으로, 스스로 이를 정리합니다:

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

전체 코드를 확인하려면 [여기](https://jsfiddle.net/09jvu65m/)를 확인하세요. 만일 스스로가 단일 컴포넌트를 셋업하고 제거하는 과정을 여러 번 반복하고 있는 경우, 모듈화된 컴포넌트가 가장 좋은 솔루션일 수 있다는 사실을 기억해 두세요. 위의 경우에는 재사용 가능한 `<input-datepicker>` 컴포넌트를 만드는 것을 추천합니다.

프로그래밍적 이벤트 리스너에 대해서 더 학습하고 싶다면 API 문서의 [Events Instance Methods](https://vuejs.org/v2/api/#Instance-Methods-Events) 를 확인 해 보세요.



<p class="tip"> Vue의 이벤트 시스템은 브라우저의 <a href="https://developer.mozilla.org/en-US/docs/Web/API/EventTarget">EventTarget API</a>와는 다르다는 사실을 기억하세요. 비슷하게 동작하기는 하지만, <code>$emit</code>, <code>$on</code>, and <code>$off</code> 는 <code>dispatchEvent</code>, <code>addEventListener</code>, and <code>removeEventListener</code>의 축약어가 <strong>아닙니다.</strong></p>

## 순환 참조

### 재귀 컴포넌트

컴포넌트는 재귀적으로 템플릿 안에서 호출될 수 있습니다. 하지만 `name` 옵션을 이용해서만 호출될 수 있습니다:

``` js
name: 'unique-name-of-my-component'
```

`Vue.component`를 이용해 컴포넌트를 전역으로 등록하는 경우, 전역 ID는 자동으로 컴포넌트의 `name` 옵션의 값으로 설정됩니다.

``` js
Vue.component('unique-name-of-my-component', {
  // ...
})
```

주의하지 않으면 재귀 컴포넌트는 무한루프를 발생시킬 수도 있습니다.

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

위와 같은 컴포넌트는 "max stack size exceeded(최대 스택 사이즈가 초과되었습니다)" 에러를 출력하므로, 재귀적 호출에 올바른 조건이 설정되어있는지 확인하여야 합니다. (예: `v-if` 에 최종적으로 `false`가 들어가는가?)

### 두 컴포넌트 사이의 순환 참조

Finder나 File Explorer 같은 파일 디렉토리 트리를 만드는 경우를 생각해 봅시다. 아마 아래와 같은 `tree-foler` 컴포넌트를 만들었을 것입니다:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

그리고 `tree-folder-contents` 컴포넌트는 아래와 같은 템플릿을 가지고 있을 것입니다:

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

자세히 보면 이 컴포넌트들이 실제로는 서로의 후손이면서 조상이라는 것을 알 수 있습니다 - 패러독스죠! 만약 당신이 `Vue.component`를 이용해서 컴포넌트를 전역으로 등록하는 경우에 이 패러독스는 자연스럽게 해결됩니다. 이렇게 문제가 해결되었다면 여기까지만 읽어도 괜찮습니다.

하지만 만약에 **모듈 시스템**을 이용해(즉, Webpack이나 Browserify를 이용해) require 혹은 import를 시도한 경우, 아래와 같은 에러가 발생합니다:

```
Failed to mount component: template or render function not defined.
```

무슨 일이 일어났는지를 설명하기 위해서, 우리의 컴포넌트를 A와 B라고 부르겠습니다. 모듈 시스템의 입장에서는 A가 필요한데 A는 B가 필요하고, 다시 B는 A가 필요하고, 하지만 A는 B가 필요하고, 기타 많은 것들(...)이 필요하게 됩니다. 즉, 두 컴포넌트 모두 다른 컴포넌트 하나가 정의되기 전에 정의될 수 없는 루프에 빠지는 것입니다. 이를 해결하기 위해서는 모듈 시스템에 "결과적으로 A는 B를 필요로 하게 되지만, B를 우선적으로 정의할 필요는 없다"라는 사실을 전달하여야 합니다.

본 예시에서는 `tree-folder` 컴포넌트에 포인트를 만들어 봅시다.  `tree-folder-component` 컴포넌트가 패러독스를 발생시키는 자식요소라는 것을 알고 있으므로, `beforeCreate` 라이프사이클 훅이 호출되기를 기다렸다가 컴포넌트를 등록합니다:

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
}
```

또는 컴포넌트를 지역적으로 등록할 때 Webpack의 비동기 `import` 를 사용하는 방법도 있습니다:

``` js
components: {
  TreeFolderContents: () => import('./tree-folder-contents.vue')
}
```

문제가 해결되었습니다!

## 템플릿을 정의하는 다른 방법

### 인라인 템플릿

특수한 속성인 `inline-template`가 자식 컴포넌트에 존재하는 경우, 컴포넌트는 이를 분리된 컨텐츠로 보지 않고 현재 템플릿 안에 있는 컨텐츠로 취급합니다. 이는 좀 더 유연한 템플릿 설계가 가능하게 합니다.

``` html
<my-component inline-template>
  <div>
    <p>이는 컴포넌트 자신의 템플릿으로써 컴파일되었습니다.</p>
    <p>부모에 인용된 컨텐츠가 아닙니다.</p>
  </div>
</my-component>
```

인라인 템플릿은 Vue가 연결된 DOM 엘리먼트 내부에 정의되어야 합니다.

<p class="tip"><code>inline-template</code>을 사용하게 되면 템플릿의 스코프를 쉽게 파악할 수 없게 됩니다. <code>template</code> 옵션을 사용하거나 .vue 파일 안에 있는 <code>&lt;template&gt;</code> 엘리먼트 내부에 템플릿을 정의하는 방법을 권장합니다.</p>

### X-Templates

템플릿을 스크립트 엘리먼트 안에 정의하는 또다른 방법으로써, `text/x-template` 타입을 이용해 템플릿을 id로 참조할 수 있습니다. 예를 들어:

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

작성한 x-template는 Vue가 연결된 DOM 엘리먼트의 바깥에서 정의되어야 합니다.

<p class="tip">이러한 방법은 커다란 템플릿의 데모나 매우 작은 어플리케이션에 유용할 수는 있지만, 템플릿과 컴포넌트의 나머지 부분들을 분리시키기 때문에 가급적이면 피하는 것이 좋습니다.</p>

## 업데이트 제어

Vue의 반응형 시스템 덕분에, (제대로 사용했다면) 업데이트 될 타이밍을 항상 알 수 있습니다. 하지만 반응형 데이터가 변경되지 않았음에도 예외적으로 컴포넌트를 강제 업데이트 해야하는 예외적인 경우들이 있습니다. 반대로, 불필요한 업데이트를 방지해야 하는 경우도 있을 수 있습니다.

### 업데이트 강제하기

<p class="tip">만약에 Vue에서 강제 업데이트를 시도하고 계신다면, 99.99%의 경우는 어딘가 잘못된 것입니다</p>

[배열](https://vuejs.org/v2/guide/list.html#Caveats) 이나 [오브젝트](https://vuejs.org/v2/guide/list.html#Object-Change-Detection-Caveats)를 이용한 반응형 시스템에 변경 감지 주의사항을 설정하지 않았거나, `data` 와 같은 뷰의 반응형 시스템이 추적하지 못하는 상태에 의존하고 있는 경우가 있습니다.

하지만 극히 드문 경우로써, 위의 경우에 해당하지 않지만 데이터를 강제로 업데이트 해야 하는 경우, [`$forceUpdate`](../api/#vm-forceUpdate)를 사용할 수 있습니다.

### `v-once`를 사용하는 정적 컴포넌트

Vue는 순수한 HTML 엘리먼트를 아주 빠르게 렌더링할 수 있지만, 간혹 컴포넌트가 **많은** 정적 콘텐츠를 가지고 있을 수 있습니다. 이 경우, 루트 엘리먼트에 `v-once` 디렉티브를 적용하여 한 번 렌더링된 후 캐싱되도록 할 수 있습니다.

``` js
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... 수많은 정적 컨텐츠들 ...
    </div>
  `
})
```

<p class="tip"> 다시 한번 강조하지만, 이러한 패턴을 남용하지 마세요. 이는 렌더링할 정적 컨텐츠가 굉장히 많은 경우에 편리하게 사용할 수는 있지만, 느리게 렌더링 되는 것을 인지하지 못할 정도라면 필수적이지 않습니다. -- 더해서, 이런 방식은 추후에 많은 혼란을 야기할 수 있습니다. 예를 들어, <code>v-once</code>에 친숙하지 않은 개발자이거나 실수로 놓치는 경우에 그들은 템플릿이 정상적으로 업데이트 되지 않는 문제에 대해서 많은 시간을 소비하게 될 수 있습니다.</p>