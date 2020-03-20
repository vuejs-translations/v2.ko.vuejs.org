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

아래 결괴를 확인하세요. :

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

이제 _Posts_ 탭은 (게시물이 선택된) 상태를 유지할 수 있고 심지어 렌더링하지 않습니다. 완성된 코드는 [this fiddle](https://jsfiddle.net/chrisvfritz/Lp20op9o/)에서 볼 수 있습니다.

<p class="tip">`<keep-alive>`가 컴포넌트에서 `name` 옵션을 사용하거나 로컬/글로벌 등록을 하여 정의된 모든 것들로 전환되고 있는 컴포넌트들을 요구한다는 것을 유의하세요.</p>

더 자세한 내용은 [API reference](../api/#keep-alive)에서 `<keep-alive>` 확인해주세요.

## Async Components

<div class="vueschool"><a href="https://vueschool.io/lessons/dynamically-load-components?friend=vuejs" target="_blank" rel="sponsored noopener" title="Free Vue.js Async Components lesson">Watch a free video lesson on Vue School</a></div>

In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it's needed. To make that easier, Vue allows you to define your component as a factory function that asynchronously resolves your component definition. Vue will only trigger the factory function when the component needs to be rendered and will cache the result for future re-renders. For example:

```js
Vue.component("async-example", function(resolve, reject) {
  setTimeout(function() {
    // Pass the component definition to the resolve callback
    resolve({
      template: "<div>I am async!</div>"
    });
  }, 1000);
});
```

As you can see, the factory function receives a `resolve` callback, which should be called when you have retrieved your component definition from the server. You can also call `reject(reason)` to indicate the load has failed. The `setTimeout` here is for demonstration; how to retrieve the component is up to you. One recommended approach is to use async components together with [Webpack's code-splitting feature](https://webpack.js.org/guides/code-splitting/):

```js
Vue.component("async-webpack-example", function(resolve) {
  // This special require syntax will instruct Webpack to
  // automatically split your built code into bundles which
  // are loaded over Ajax requests.
  require(["./my-async-component"], resolve);
});
```

You can also return a `Promise` in the factory function, so with Webpack 2 and ES2015 syntax you can do:

```js
Vue.component(
  "async-webpack-example",
  // The `import` function returns a Promise.
  () => import("./my-async-component")
);
```

When using [local registration](components-registration.html#Local-Registration), you can also directly provide a function that returns a `Promise`:

```js
new Vue({
  // ...
  components: {
    "my-component": () => import("./my-async-component")
  }
});
```

<p class="tip">If you're a <strong>Browserify</strong> user that would like to use async components, its creator has unfortunately [made it clear](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224) that async loading "is not something that Browserify will ever support." Officially, at least. The Browserify community has found [some workarounds](https://github.com/vuejs/vuejs.org/issues/620), which may be helpful for existing and complex applications. For all other scenarios, we recommend using Webpack for built-in, first-class async support.</p>

### Handling Loading State

> New in 2.3.0+

The async component factory can also return an object of the following format:

```js
const AsyncComponent = () => ({
  // The component to load (should be a Promise)
  component: import("./MyComponent.vue"),
  // A component to use while the async component is loading
  loading: LoadingComponent,
  // A component to use if the load fails
  error: ErrorComponent,
  // Delay before showing the loading component. Default: 200ms.
  delay: 200,
  // The error component will be displayed if a timeout is
  // provided and exceeded. Default: Infinity.
  timeout: 3000
});
```

> Note that you must use [Vue Router](https://github.com/vuejs/vue-router) 2.4.0+ if you wish to use the above syntax for route components.
