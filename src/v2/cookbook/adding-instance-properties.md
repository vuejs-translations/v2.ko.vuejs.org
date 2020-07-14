---
title: 인스턴스 프로퍼티 추가하기
type: cookbook
order: 2
---

## 기본예제

당신은 많은 컴포넌트에서 사용하고 싶을 데이터나 유틸리티들이 있을 테지만  [global scope에 의해 오염되는 것](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/ch3.md)은 원하지 않을 것입니다. 이 때, 이것들을 prototype에서 정의하면 각 Vue instance에서 사용할 수 있습니다.

```js
Vue.prototype.$appName = 'My App'
```

이제 `$appName` 은 모든 Vue instance에서 사용이 가능합니다. 심지어 아직 만들어지지 않았더라도요. 다음을 실행해 보세요:

```js
new Vue({
  beforeCreate: function() {
    console.log(this.$appName)
  }
})
```

`"My App"` 이 콘솔에 찍힐 것입니다!

## 인스턴스 프로퍼티 스코프의 중요성

당신은 아마 이걸 궁금해 할 수도 있을 겁니다:

> “왜 appName 앞에 $가 붙지? 이거 중요한건가? 이게 왜 있는건데?“

어렵지 않습니다. `$`는 모든 인스턴스에서 사용 가능한 프로퍼티라고 알려주는 뷰에서 사용하는 언어입니다. 이것은 이미 정의된 데이터나, computed 요소, 메소드와의 충돌을 예방합니다.

> “충돌? 무슨 소리인가요?”

또 다른 좋은 질문입니다! 만약 당신이:

```js
Vue.prototype.appName = 'My App'
```

 라고 설정했다면 어떤 로그가 찍힐까요?

```js
new Vue({
  data: {
    // 이런, appName 우리가 정의한
    // 인스턴스 프로퍼티와 같은 이름이에요!
    appName: 'The name of some other app'
  },
  beforeCreate: function() {
    console.log(this.appName)
  },
  created: function() {
    console.log(this.appName)
  }
})
```

정답은 "MyApp"입니다. 그리고 "The name of some other app"입니다. 그 이유는, `this.appName`은 인스턴스가 생성될 `data`에 의해 [덮어쓰기](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/this-object-prototypes/ch5.md) 되었기 때문입니다. 그래서 우리는 이것을 피하기 위해 $를 사용합니다. 심지어 원한다면 플러그인이나 feature와의 충돌을 피하기 위해 `$_appName`나 `ΩappName`과 같은 자신만의 표현을 사용할 수 있습니다.
 

## 진짜 일어나는 일들: Vue 리소스를 Axios로 바꿔치기

[이제 안쓰일 리소스](https://medium.com/the-vue-point/retiring-vue-resource-871a82880af4)를 바꿔야 한다고 가정해 봅시다. 당신은 `this.$http` 를 통해 리퀘스트에 접속하는 걸 자주 사용했는데, axios를 통해같은 동작을 하게 하고 싶다면,

그냥 다음 구문을 코드에 삽입하면 됩니다:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.15.2/axios.js"></script>

<div id="app">
  <ul>
    <li v-for="user in users">{{ user.name }}</li>
  </ul>
</div>
```

axios를 `Vue.prototype.$http` 라고 지정(별칭)합니다.

```js
Vue.prototype.$http = axios
```

이렇게 하면 당신은 어떤 뷰 인스턴스 내에서도 이 메소드를 `this.$http.get`와 동일하게 사용할 수 있습니다:

```js
new Vue({
  el: '#app',
  data: {
    users: []
  },
  created() {
    var vm = this
    this.$http
      .get('https://jsonplaceholder.typicode.com/users')
      .then(function(response) {
        vm.users = response.data
      })
  }
})
```

## 프로토타입 메소드의 구문

눈치채지 못했을 수도 있지만, 자바스크립트 프로토타입에 더해진 메소드는 인스턴스의 context를 얻을 수 있습니다. 그 뜻은 이것이 접근 가능한 데이터, computed 요소, 메소드 또는 인스턴스에 정의된 어떤 것으로도 사용 가능하다는 의미입니다.

이제 이 `$reverseText`로의 모험을 떠나봅시다.

```js
Vue.prototype.$reverseText = function(propertyName) {
  this[propertyName] = this[propertyName]
    .split('')
    .reverse()
    .join('')
}

new Vue({
  data: {
    message: 'Hello'
  },
  created: function() {
    console.log(this.message) // => "Hello"
    this.$reverseText('message')
    console.log(this.message) // => "olleH"
  }
})
```

만약 당신이 ES6/2015의 화살표 함수를 사용하고 있다면 내부적으로 부모 스코프에 바인딩되기 때문에, context 바인딩이 먹히지 **않을** 것이라는 것을 주의합시다.
즉, 이렇게 화살표 함수를 사용하면:

```js
Vue.prototype.$reverseText = propertyName => {
  this[propertyName] = this[propertyName]
    .split('')
    .reverse()
    .join('')
}
```

다음과 같은 에러를 발생할 것입니다:

```log
Uncaught TypeError: Cannot read property 'split' of undefined
```

## 이 패턴을 피하기 위해

당신이 프로토타입 요소의 스코프에 충분히 주의만 한다면, 이 패턴을 사용하는 일은 꽤나 안전할 것입니다. - 말 그대로, 버그가 생기는 것은 드뭅니다.

하지만, 다른 개발자에겐 종종 혼란이 일어날 수 있습니다. 그 사람은, 예를들어, `$http` 를 보고 "오, 이 뷰 feature는 몰랐는데!"하고 생각할 수 있습니다. 그리고 다른 프로젝트에 가서 `$http`가 정의되지 않은 것을 보고 헷갈려 할 겁니다. 아마 구글링을 해서 답을 찾아보려고 할 수도 있지만, 원하는 결과를 찾지는 못할겁니다. 왜냐하면, 사실 그건 별칭(alias) 된 axios이기 때문이죠.

**명시성에는 댓가를 따릅니다.** 컴포넌트를 봤을 때, `$http` 가 어디에서 비롯된 건지 알려주는 건 아무것도 없습니다. 뷰에서 직접? 플러그인을 통해? 협업자가?

대안이 대체 뭘까요?

## 대체 패턴

### 모듈 시스템을 사용하지 않는다면

Webpack나 Browserify와 같은 모듈 시스템을 사용하지 **않는** 어플리케이션이라면, _어떤_ 향상된 자바스크립트 프론트엔드에서도 자주 사용되는 패턴이 있습니다: 글로벌 `App` 오브젝트라는 패턴입니다.

만약 당신이 더하고 싶은 것이 뷰를 통해 뭔가 특별한 것을 하지 않는다면, 이것은 좋은 대체제가 될 것입니다. 예를 들어봅시다:

```js
var App = Object.freeze({
  name: 'My App',
  version: '2.1.4',
  helpers: {
    // This is a purely functional version of
    // the $reverseText method we saw earlier
    reverseText: function(text) {
      return text
        .split('')
        .reverse()
        .join('')
    }
  }
})
```

<p class="tip">만약 당신이 이 Object.freeze 부분에서 고개를 갸우뚱 한다면, 이것은 object가 미래에 바뀌는 것을 방지하는 기능을 합니다. 이것은 기본적으로 가지고 있는 모든 프로퍼티를 지속되게 하고, 미래의 상태(state) 버그를 예방합니다.</p>

이제 공유된 프로퍼티의 원천이 조금 더 명확해 졌습니다: 앱에는 `App` 으로 정의된 오브젝트가 있습니다. 이것을 찾고 싶다면, 개발자는 프로젝트 전체 검색을 하면 됩니다.

App 을 사용하는 또다른 장점은 이것은 이제 뷰와 관련있든(Vue-related) 없든 당신의 코드 _어디에서든_ 사용 가능하다는 점입니다. 이것은 함수의 `this` 안에 있는 프로퍼티에 접근해야 하는 것보다 인스턴스 옵션의 value에 바로 추가할 수 있다는 의미를 포함한다는 소리입니다.

```js
new Vue({
  data: {
    appVersion: App.version
  },
  methods: {
    reverseText: App.helpers.reverseText
  }
})
```

### 모듈 시스템을 사용한다면

모듈 시스템에 접속하면, 공유된 코드를 모듈로 파악하는 것은 쉽습니다. 그리고 `require`/`import`를 사용하여 필요한 어느곳에든 사용할 수 있습니다. 이것은 명시성의 전형입니다. 왜냐하면, 각 파일에서 당신은 의존성의 리스트를 얻게 됩니다. 당신은 그것이 어디에서 왔는지 _정확하게_ 알 수 있습니다.

좀 복잡해지겠지만, 이 방법을 사용하면 확실하게 유지보수가 수월해집니다. 특히, 다른 개발자와 협업하거나 큰 앱을 개발할 때 말이죠.
