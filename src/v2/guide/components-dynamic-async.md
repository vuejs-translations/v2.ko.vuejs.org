---
title: 동적 & 비동기 컴포넌트
type: guide
order: 105
---

> 이 페이지는 여러분이 이미 [컴포넌트 기초](components.html)를 읽었다고 가정하고 쓴 내용입니다. 컴포넌트가 처음이라면 기초 문서를 먼저 읽으시기 바랍니다.

## `keep-alive` 동적 컴포넌트

초기에는, 탭 인터페이스에서 컴포넌트들을 전환하기 위해서 `is` 특성을 사용했습니다. :

{% codeblock lang:html %}
<component v-bind:is="currentTabComponent"></component>
{% endcodeblock %}

컴포넌트들을 전환할 때 가끔 성능상의 이유로 상태를 유지하거나 재-렌더링을 피하길 원할 수 있습니다. 예를 들면, 탭 인터페이스를 약간 확장 할 때 :

{% raw %}

<div id="dynamic-component-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    v-bind:class="['dynamic-component-demo-tab-button', { 'dynamic-component-demo-active': currentTab === tab }]"
    v-on:click="currentTab = tab"
  >{{ tab }}</button>
  <component
    v-bind:is="currentTabComponent"
    class="dynamic-component-demo-tab"
  ></component>
</div>
<script>
Vue.component('tab-posts', {
  data: function () {
    return {
      posts: [
        {
          id: 1,
          title: 'Cat Ipsum',
          content: '<p>Dont wait for the storm to pass, dance in the rain kick up litter decide to want nothing to do with my owner today demand to be let outside at once, and expect owner to wait for me as i think about it cat cat moo moo lick ears lick paws so make meme, make cute face but lick the other cats. Kitty poochy chase imaginary bugs, but stand in front of the computer screen. Sweet beast cat dog hate mouse eat string barf pillow no baths hate everything stare at guinea pigs. My left donut is missing, as is my right loved it, hated it, loved it, hated it scoot butt on the rug cat not kitten around</p>'
        },
        {
          id: 2,
          title: 'Hipster Ipsum',
          content: '<p>Bushwick blue bottle scenester helvetica ugh, meh four loko. Put a bird on it lumbersexual franzen shabby chic, street art knausgaard trust fund shaman scenester live-edge mixtape taxidermy viral yuccie succulents. Keytar poke bicycle rights, crucifix street art neutra air plant PBR&B hoodie plaid venmo. Tilde swag art party fanny pack vinyl letterpress venmo jean shorts offal mumblecore. Vice blog gentrify mlkshk tattooed occupy snackwave, hoodie craft beer next level migas 8-bit chartreuse. Trust fund food truck drinking vinegar gochujang.</p>'
        },
        {
          id: 3,
          title: 'Cupcake Ipsum',
          content: '<p>Icing dessert soufflé lollipop chocolate bar sweet tart cake chupa chups. Soufflé marzipan jelly beans croissant toffee marzipan cupcake icing fruitcake. Muffin cake pudding soufflé wafer jelly bear claw sesame snaps marshmallow. Marzipan soufflé croissant lemon drops gingerbread sugar plum lemon drops apple pie gummies. Sweet roll donut oat cake toffee cake. Liquorice candy macaroon toffee cookie marzipan.</p>'
        }
      ],
      selectedPost: null
    }
  },
  template: '\
    <div class="dynamic-component-demo-posts-tab">\
      <ul class="dynamic-component-demo-posts-sidebar">\
        <li\
          v-for="post in posts"\
          v-bind:key="post.id"\
          v-bind:class="{ \'dynamic-component-demo-active\': post === selectedPost }"\
          v-on:click="selectedPost = post"\
        >\
          {{ post.title }}\
        </li>\
      </ul>\
      <div class="dynamic-component-demo-post-container">\
        <div \
          v-if="selectedPost"\
          class="dynamic-component-demo-post"\
        >\
          <h3>{{ selectedPost.title }}</h3>\
          <div v-html="selectedPost.content"></div>\
        </div>\
        <strong v-else>\
          Click on a blog title to the left to view it.\
        </strong>\
      </div>\
    </div>\
  '
})
Vue.component('tab-archive', {
  template: '<div>Archive component</div>'
})
new Vue({
  el: '#dynamic-component-demo',
  data: {
    currentTab: 'Posts',
    tabs: ['Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
</script>
<style>
.dynamic-component-demo-tab-button {
  padding: 6px 10px;
  border-top-left-radius: 3px;
  border-top-right-radius: 3px;
  border: 1px solid #ccc;
  cursor: pointer;
  background: #f0f0f0;
  margin-bottom: -1px;
  margin-right: -1px;
}
.dynamic-component-demo-tab-button:hover {
  background: #e0e0e0;
}
.dynamic-component-demo-tab-button.dynamic-component-demo-active {
  background: #e0e0e0;
}
.dynamic-component-demo-tab {
  border: 1px solid #ccc;
  padding: 10px;
}
.dynamic-component-demo-posts-tab {
  display: flex;
}
.dynamic-component-demo-posts-sidebar {
  max-width: 40vw;
  margin: 0 !important;
  padding: 0 10px 0 0 !important;
  list-style-type: none;
  border-right: 1px solid #ccc;
}
.dynamic-component-demo-posts-sidebar li {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  cursor: pointer;
}
.dynamic-component-demo-posts-sidebar li:hover {
  background: #eee;
}
.dynamic-component-demo-posts-sidebar li.dynamic-component-demo-active {
  background: lightblue;
}
.dynamic-component-demo-post-container {
  padding-left: 10px;
}
.dynamic-component-demo-post > :first-child {
  margin-top: 0 !important;
  padding-top: 0 !important;
}
</style>
{% endraw %}

게시물을 선택하고, _Archive_ 탭으로 전환하고, 다시 *Posts*로 전환할 때, 선택했던 게시물이 더는 보지 않는 것 알아차릴 수 있습니다. 그 이유는 매번 새로운 탭을 선택할 때, Vue는 `currentTabComponent`의 새로운 인스턴스를 생성하기 때문입니다.

동적 컴포넌트를 재생성하는 것은 보통은 유용한 동작입니다. 하지만 이 경우에는, 탭 컴포넌트 인스턴스가 처음 생성될 때 캐시 되는 것을 선호합니다. 이런 문제를 해결하기 위해서, 동적 컴포넌트를 `<keep-alive>` 엘리먼트로 둘러쌀 수 있습니다. :

```html
<!-- Inactive components will be cached! -->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

아래 결과를 확인하세요. :

{% raw %}

<div id="dynamic-component-keep-alive-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    v-bind:class="['dynamic-component-demo-tab-button', { 'dynamic-component-demo-active': currentTab === tab }]"
    v-on:click="currentTab = tab"
  >{{ tab }}</button>
  <keep-alive>
    <component
      v-bind:is="currentTabComponent"
      class="dynamic-component-demo-tab"
    ></component>
  </keep-alive>
</div>
<script>
new Vue({
  el: '#dynamic-component-keep-alive-demo',
  data: {
    currentTab: 'Posts',
    tabs: ['Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
</script>
{% endraw %}

이제 _Posts_ 탭은 (게시물이 선택된) 상태를 유지할 수 있고 다시 렌더링하지도 않습니다. 완성된 코드는 [this fiddle](https://jsfiddle.net/chrisvfritz/Lp20op9o/)에서 볼 수 있습니다.

<p class="tip">`<keep-alive>`가 컴포넌트에서 `name` 옵션을 사용하거나 로컬/글로벌 등록을 하여 정의된 모든 것들로 전환되고 있는 컴포넌트들을 요구한다는 것에 유의하세요.</p>

더 자세한 내용은 [API reference](../api/#keep-alive)에서 `<keep-alive>` 항목을 확인해주세요.





## 비동기 컴포넌트

<div class="vueschool"><a href="https://vueschool.io/lessons/dynamically-load-components?friend=vuejs" target="_blank" rel="sponsored noopener" title="Free Vue.js Async Components lesson">Watch a free video lesson on Vue School</a></div>
커다란 어플리케이션의 경우, 앱을 작은 조각들로 나누어 두고 필요한 경우에만 서버로부터 필요한 컴포넌트를 로드해야 할 수 있습니다. 이런 작업을 쉽게 할 수 있도록 Vue는 팩토리 함수를 이용해 컴포넌트를 비동기적으로 정의하는 것을 허용합니다. Vue는 컴포넌트가 필요할 때 팩토리 함수를 실행하고 미래를 위해 결과를 캐시합니다. 예를 들어:

```js
Vue.component("async-example", function(resolve, reject) {
  setTimeout(function() {
    // 컴포넌트 정의를 resolve 콜백을 통해 전달
    resolve({
      template: "<div>I am async!</div>"
    });
  }, 1000);
});
```

위의 예에서 보다시피 팩토리 함수는 `resolve` 콜백을 받습니다. 콜백은 서버에 컴포넌트의 정의를 요청했을 때 호출됩니다. 물론 `reject(reason)` 을 호출하여 로딩이 실패한 경우에 대응할 수도 있습니다. setTimeout은 예제일 뿐이며, 컴포넌트를 어떻게 받아올지는 원하는 방식을 선택하면 됩니다. 권장되는 접근 방식으로써, [Webpack's code-splitting feature](https://webpack.js.org/guides/code-splitting/) 와 함께 비동기 컴포넌트를 사용하는 패턴이 있습니다:

```js
Vue.component("async-webpack-example", function(resolve) {
  // 아래의 특별한 'require' 문법은 
  // 웹팩이 Ajax를 통해 로드한 번들들을 이용해
  // 코드를 자동으로 분할하도록 합니다.
  require(["./my-async-component"], resolve);
});
```

팩토리 함수 안에서 `Promise`를 반환할 수도 있습니다. Webpack 2와 ES2015 문법을 이용해 아래와 같이 쓸 수 있습니다:

```js
Vue.component(
  "async-webpack-example",
  // `import` 함수는 Promise를 반환합니다.
  () => import("./my-async-component")
);
```

[local registration](components-registration.html#Local-Registration)을 이용하면, `Promise`를 반환하는 함수를 직접 작성할 수도 있습니다:

```js
new Vue({
  // ...
  components: {
    "my-component": () => import("./my-async-component")
  }
});
```

<p class="tip">만약에 <strong>Browserify</strong>를 이용해서 비동기 컴포넌트를 구현하고자 하는 경우, 불행히도 Browserify를 만든 개발자가 "비동기 컴포넌트는 Browserify가 지원할 계회기 전혀 없는 것이다" 라고 이야기했다는 사실을 알아야 합니다. ([made it clear](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224))<br/>
  Browserify 커뮤니티는 이미 존재하는 복잡한 예제에 대한 [대안](https://github.com/vuejs/vuejs.org/issues/620)을 찾긴 했습니다. 가능한 모든 시나리오에 대해 비동기를 완벽히 지원하는 웹팩을 이용하는 것을 권장합니다.</p>

### Handling Loading State

> 2.3.0+ 에서 추가

비동기 컴포넌트 팩토리는 아래와 같은 포맷으로 오브젝트를 반환할 수도 있습니다:

```js
const AsyncComponent = () => ({
  // 로드 할 컴포넌트(Promise여야 합니다.)
  component: import("./MyComponent.vue"),
  // 비동기 컴포넌트가 로딩중일 때 사용할 컴포넌트
  loading: LoadingComponent,
  // 비동기 컴포넌트 로딩이 실패했을 때 사용할 컴포넌트
  error: ErrorComponent,
  // 로딩 컴포넌트를 보여주기 전의 지연시간. (기본값: 200ms)
  delay: 200,
  // 초과되었을 때 에러 컴포넌트를 표시할 타임아웃. (기본값: 무한대)
  timeout: 3000
});
```

> 위에 작성된 문법을 라우터 컴포넌트에 사용하기 위해서는 2.4.0+ 버전의 [Vue Router](https://github.com/vuejs/vue-router) 를 사용해야 한다는 것을 기억하세요!
