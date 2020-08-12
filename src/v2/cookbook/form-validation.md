---
title: Form Validation
type: cookbook
order: 3
---

## 기본예제

<div class="vueschool"><a href="https://vueschool.io/lessons/vuejs-form-validation-diy?friend=vuejs" target="_blank" rel="sponsored noopener" title="Free Vue.js Form Validation Lesson">Watch a free lesson on Vue School</a></div>

Form 검증은 브라우저에서 네이티브로 지원하지만, 서로 다른 브라우저의 방식의 차이로 이것을 사용할 때 주의가 필요합니다. 설령 검증이 완벽하게 지원된다 하더라도 커스텀 검증이나 더 자세한 메뉴얼이 필요한 경우가 뷰 기반의 해결방안이 차라리 나을 수 있습니다. 간단한 예제로 살펴봅시다.

세가지 필드를 가진 form이 주어지고, 두개를 요청했습니다. HTML을 먼저 보겠습니다:

``` html
<form
  id="app"
  @submit="checkForm"
  action="https://vuejs.org/"
  method="post"
>

  <p v-if="errors.length">
    <b>Please correct the following error(s):</b>
    <ul>
      <li v-for="error in errors">{{ error }}</li>
    </ul>
  </p>

  <p>
    <label for="name">Name</label>
    <input
      id="name"
      v-model="name"
      type="text"
      name="name"
    >
  </p>

  <p>
    <label for="age">Age</label>
    <input
      id="age"
      v-model="age"
      type="number"
      name="age"
      min="0"
    >
  </p>

  <p>
    <label for="movie">Favorite Movie</label>
    <select
      id="movie"
      v-model="movie"
      name="movie"
    >
      <option>Star Wars</option>
      <option>Vanilla Sky</option>
      <option>Atomic Blonde</option>
    </select>
  </p>

  <p>
    <input
      type="submit"
      value="Submit"
    >
  </p>

</form>
```

제일 위부터 살펴봅시다. `<form>` 태그는 우리가 Vue 컴포넌트로 사용할 수 있는 id를 가지고 있습니다. 그 아래에 submit 핸들러가 있고, `action` 은 실제 서버의 (당신이 백업 서버사이드 검증을 가지고 있는) 어떠한 부분을 가리키는 비영구적인 url을 담고 있습니다. 

그보다 더 아래 `p` 는 에러 상태에 따라 보여주거나 숨겨집니다. 이것은 form 위에 간단한 에러들의 리스트를 랜더링 할 것입니다. 또한 우리는 모든 필드가 채워진 후에야 submit 검증을 할 것임을 기억하세요.

마지막으로 기억해야 할 건, 각 세개의 필드는 `v-model` 을 통해 value를 대응시켜 자바스크립트에서 동작하도록 해준다는 것입니다. 이제 확인해 봅시다.

``` js
const app = new Vue({
  el: '#app',
  data: {
    errors: [],
    name: null,
    age: null,
    movie: null
  },
  methods:{
    checkForm: function (e) {
      if (this.name && this.age) {
        return true;
      }

      this.errors = [];

      if (!this.name) {
        this.errors.push('Name required.');
      }
      if (!this.age) {
        this.errors.push('Age required.');
      }

      e.preventDefault();
    }
  }
})
```

Fair충분히 짧고 심플합니다. 우린 정렬을 사용해 에러를 보존하게 하고 세 폼 필드에 `null` 값을 세팅해 주었습니다. 영화는 선택 사항이기 때문에 checkForm 로직은 (submit에서 돌아간다는 점 기억해 두세요) 이름과 나이만을 체크합니다. 아래 데모를 통해 확인 가능합니다. 성공적인 전송을 위해서는 POST를 사용해야 한다는 점 잊지 마세요.

<p data-height="265" data-theme-id="0" data-slug-hash="GObpZM" data-default-tab="html,result" data-user="cfjedimaster" data-embed-version="2" data-pen-title="form validation 1" class="codepen">See the Pen <a href="https://codepen.io/cfjedimaster/pen/GObpZM/">form validation 1</a> by Raymond Camden (<a href="https://codepen.io/cfjedimaster">@cfjedimaster</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 커스텀 검증 사용하기

두 번째 예시로, 두번째 필드(나이)를 약간의 커스텀 검증을 사용한 이메일로 변경해 보겠습니다. 코드는 스텍오버플로우의 [How to validate email address in JavaScript?](https://stackoverflow.com/questions/46155/how-to-validate-email-address-in-javascript)에서 가지고 왔습니다. 이건 정말 굉장한 질문입니다. 어느정도나면 당신이 페이스북에 남기는 가장 격렬한 정치적/종교적 토론조차 누가 세상에서 제일 맛있는 맥주를 만드는지에 관한 약간의 불일치로 보일 정도입니다. 진짜로요. 짱이죠. 여기에 HTML이 있습니다, 거의 첫번째 예시와 다를 바가 없지만요.

``` html
<form
  id="app"
  @submit="checkForm"
  action="https://vuejs.org/"
  method="post"
  novalidate="true"
>

  <p v-if="errors.length">
    <b>Please correct the following error(s):</b>
    <ul>
      <li v-for="error in errors">{{ error }}</li>
    </ul>
  </p>

  <p>
    <label for="name">Name</label>
    <input
      id="name"
      v-model="name"
      type="text"
      name="name"
    >
  </p>

  <p>
    <label for="email">Email</label>
    <input
      id="email"
      v-model="email"
      type="email"
      name="email"
    >
  </p>

  <p>
    <label for="movie">Favorite Movie</label>
    <select
      id="movie"
      v-model="movie"
      name="movie"
    >
      <option>Star Wars</option>
      <option>Vanilla Sky</option>
      <option>Atomic Blonde</option>
    </select>
  </p>

  <p>
    <input
      type="submit"
      value="Submit"
    >
  </p>

</form>
```

변화는 적지만, 맨 위에 `novalidate="true"` 는 주목해 주세요. 이게 중요한 이유는 `type="email"`를 필드에서 사용하면 브라우저에서 검증을 해주기 때문입니다. 솔직히 말하자면 여기서는 브라우저의 검증 능력을 믿는 게 훨씬 낫지만, 커스텀 검증을 예시로 사용해야하기 때문에 이것을 막을 겁니다. 여기 자바스크립트 입니다.

``` js
const app = new Vue({
  el: '#app',
  data: {
    errors: [],
    name: null,
    email: null,
    movie: null
  },
  methods: {
    checkForm: function (e) {
      this.errors = [];

      if (!this.name) {
        this.errors.push("Name required.");
      }
      if (!this.email) {
        this.errors.push('Email required.');
      } else if (!this.validEmail(this.email)) {
        this.errors.push('Valid email required.');
      }

      if (!this.errors.length) {
        return true;
      }

      e.preventDefault();
    },
    validEmail: function (email) {
      var re = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
      return re.test(email);
    }
  }
})
```

보시다시피, `validEmail`이라는 새 메소드가 추가되었고 이것은 `checkForm`이 불러옵니다. 여기 예시를 직접 볼 수 있습니다:

<p data-height="265" data-theme-id="0" data-slug-hash="vWqNXZ" data-default-tab="html,result" data-user="cfjedimaster" data-embed-version="2" data-pen-title="form validation 2" class="codepen">See the Pen <a href="https://codepen.io/cfjedimaster/pen/vWqNXZ/">form validation 2</a> by Raymond Camden (<a href="https://codepen.io/cfjedimaster">@cfjedimaster</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 커스텀 검증의 또 다른 예시

세번째 예시로는, 아마 설문조사 앱에서 본적 있었을 것을 만들어 보았습니다. 사용자는 새 행성파괴자(Star Destroyer) 모델의 feature을 작성하기 위해 위해 얼마를 소모할지 작성해야 합니다. 모든 값을 더하면 100이 되어야 합니다. 먼저 HTML입니다.

``` html
<form
  id="app"
  @submit="checkForm"
  action="https://vuejs.org/"
  method="post"
  novalidate="true"
>

  <p v-if="errors.length">
    <b>Please correct the following error(s):</b>
    <ul>
      <li v-for="error in errors">{{ error }}</li>
    </ul>
  </p>

  <p>
    Given a budget of 100 dollars, indicate how much
    you would spend on the following features for the
    next generation Star Destroyer. Your total must sum up to 100.
  </p>

  <p>
    <input
      v-model.number="weapons"
      type="number"
      name="weapons"
    > Weapons <br/>
    <input
      v-model.number="shields"
      type="number"
      name="shields"
    > Shields <br/>
    <input
      v-model.number="coffee"
      type="number"
      name="coffee"
    > Coffee <br/>
    <input
      v-model.number="ac"
      type="number"
      name="ac"
    > Air Conditioning <br/>
    <input
      v-model.number="mousedroids"
      type="number"
      name="mousedroids"
    > Mouse Droids <br/>
  </p>

  <p>
    Current Total: {{total}}
  </p>

  <p>
    <input
      type="submit"
      value="Submit"
    >
  </p>

</form>
```

먼저 input안에 5개의 다른 feature가 있다는 결 주목하세요.v-model의 .number 속성이 주는 추가 정보에 대해서도 주목해주세요. 이것은 Vue에서 해당 값을 작성할 때 숫자만 입력받아야 한다는 정보를 줍니다. 그러나 이 feature는 한가지 버그가 있는데, value가 비어있거나 한 경우 string으로 돌아가고 맙니다. 아래에 해결 방법을 적어놓았습니다. 사용자의 이해를 돕기 위해, 우리는 바로 아래에 실시간 현재 합계를 추가해 놓았습니다. 이제 자바스크립트를 봐 봅시다.

``` js
const app = new Vue({
  el: '#app',
  data:{
    errors: [],
    weapons: 0,
    shields: 0,
    coffee: 0,
    ac: 0,
    mousedroids: 0
  },
  computed: {
     total: function () {
       // must parse because Vue turns empty value to string
       return Number(this.weapons) +
         Number(this.shields) +
         Number(this.coffee) +
         Number(this.ac+this.mousedroids);
     }
  },
  methods:{
    checkForm: function (e) {
      this.errors = [];

      if (this.total != 100) {
        this.errors.push('Total must be 100!');
      }

      if (!this.errors.length) {
        return true;
      }

      e.preventDefault();
    }
  }
})
```

우리는 총합을 computed value로 총합을 표기했고, 아주 간단하게 버그를 수정했습니다. 저의 checkForm 메소드는 이제 총합이 100인지의 여부만 확인하면 됩니다. 여기를 실행해 보세요:

<p data-height="265" data-theme-id="0" data-slug-hash="vWqGoy" data-default-tab="html,result" data-user="cfjedimaster" data-embed-version="2" data-pen-title="form validation 3" class="codepen">See the Pen <a href="https://codepen.io/cfjedimaster/pen/vWqGoy/">form validation 3</a> by Raymond Camden (<a href="https://codepen.io/cfjedimaster">@cfjedimaster</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 서버사이드 검증

마지막 예시로, 우리는 ajax를 통해 서버에서 검증을 시행하는 것을 만들어 보았습니다. 폼은 당신에게 새 상품의 이름을 물을 것이고, 그 이름이 중복되지 않았는지 체크할 것입니다. 우리는 검증을 위해 [Netlify](https://www.netlify.com) 를 통해 빠른 서버리스 동작을 사용했습니다. 여기서는 중요하게 다루지 않지만, 로직은 아래와 같습니다:

``` js
function main(args) {
    return new Promise((resolve, reject) => {
        // bad product names: vista, empire, mbp
        const badNames = ['vista', 'empire', 'mbp'];

        if (badNames.includes(args.name)) {
          reject({error: 'Existing product'});
        }

        resolve({status: 'ok'});
    });
}
```

기본적으로 "vista", "empire", 그리고 "mbp"를 제외한 어떤 이름이든 사용 가능합니다. 좋아요, 이제 폼을 봅시다.

``` html
<form
  id="app"
  @submit="checkForm"
  method="post"
>

  <p v-if="errors.length">
    <b>Please correct the following error(s):</b>
    <ul>
      <li v-for="error in errors">{{ error }}</li>
    </ul>
  </p>

  <p>
    <label for="name">New Product Name: </label>
    <input
      id="name"
      v-model="name"
      type="text"
      name="name"
    >
  </p>

  <p>
    <input
      type="submit"
      value="Submit"
    >
  </p>

</form>
```

별로 중요해 보이는 게 없네요. 이제 자바스크립트로 넘어가 봅시다.

``` js
const apiUrl = 'https://openwhisk.ng.bluemix.net/api/v1/web/rcamden%40us.ibm.com_My%20Space/safeToDelete/productName.json?name=';

const app = new Vue({
  el: '#app',
  data: {
    errors: [],
    name: ''
  },
  methods:{
    checkForm: function (e) {
      e.preventDefault();

      this.errors = [];

      if (this.name === '') {
        this.errors.push('Product name is required.');
      } else {
        fetch(apiUrl + encodeURIComponent(this.name))
        .then(res => res.json())
        .then(res => {
          if (res.error) {
            this.errors.push(res.error);
          } else {
            // redirect to a new URL, or do something on success
            alert('ok!');
          }
        });
      }
    }
  }
})
```

이제 `checkForm`을 봐봅시다. 이 버전에서, 우리는 폼의 submit을 막았습니다. (다른 말이지만, 사실 뷰랑 HTML만으로도 충분하지만요.) `this.name`이 비어있는지 체크하고, api를 호출합니다. 값이 비었다면, 에러 메세지를 방출합니다. 제대로 입력했다면, (알림외에는) 아무것도 일어나지 않습니다. 하지만 URL에 포함된 제품 이름의 새 페이지로 안내하거나 다른 동작을 할 수 있습니다.  다음 데모를 확인해 보세요:

<p data-height="265" data-theme-id="0" data-slug-hash="BmgzeM" data-default-tab="js,result" data-user="cfjedimaster" data-embed-version="2" data-pen-title="form validation 4" class="codepen">See the Pen <a href="https://codepen.io/cfjedimaster/pen/BmgzeM/">form validation 4</a> by Raymond Camden (<a href="https://codepen.io/cfjedimaster">@cfjedimaster</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 대체 패턴

이 쿡북에서는 폼 검증을 "직접" 하는 것에 집중했지만, 당연히 이것을 시행해주는 좋은 뷰 라이브러리들이 있습니다. 프리패키징 라이브러리로 바꾸는 건 분명 당신의 앱 사이즈에 영향을 미치겠지만, 아마 실보다 득이 더 클 것입니다. 충분히 검증되고, 지속적으로 업데이트 되고있는 뷰 라이브러리의 예로는 다음이 포함됩니다 :

* [vuelidate](https://github.com/monterail/vuelidate)
* [VeeValidate](https://logaretm.github.io/vee-validate/)
