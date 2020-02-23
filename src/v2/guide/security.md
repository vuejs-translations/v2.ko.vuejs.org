---
title: 보안
type: guide
order: 504
---

## 취약점 보고

취약점이 보고되면, 그것을 해결하기 위해 기여자가 모든 것을 내려놓는 동시에 우리의 최고 관심사가 됩니다. 취약점을 보고하려면 [vuejs.org@gmail.com](mailto:vuejs.org@gmail.com)으로 이메일을 보내주세요.

새로운 취약점은 발견하기 드물겠지만, 항상 최신 버전의 Vue와 공식 동반 라이브러리를 사용하여 여러분의 어플리케이션을 최대한 안전하게 유지하는 것이 좋습니다.

## Vue가 여러분을 보호하기 위해 하는 일

### HTML 컨텐츠

템플릿을 사용하든 렌더 함수를 사용하든, 컨텐츠는 자동으로 이스케이프 됩니다. 다음 템플릿을 보세요.

```html
<h1>{{ userProvidedString }}</h1>
```

만약 `userProvidedString` 가 다음 값을 가지고 있다면:

```js
'<script>alert("hi")</script>'
```

그것은 다음 HTML로 이스케이프 될 것입니다.

```html
&lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;
```

그러므로 스크립트 삽입을 방지할 수 있습니다. 이 이스케이핑은 `textContent`처럼 기본 브라우저 API를 사용하므로 취약점은 오로지 브라우저 자체가 취약한 경우에만 존재할 수 있습니다.

### 속성 바인딩

마찬가지로 동적 속성 바인딩도 자동으로 이스케이프 됩니다. 다음 템플릿을 보세요.

```html
<h1 v-bind:title="userProvidedString">
  hello
</h1>
```

만약 `userProvidedString`가 다음 값을 가지고 있다면

```js
'" onclick="alert(\'hi\')'
```

그것은 다음 HTML로 이스케이프 될 것입니다.

```html
&quot; onclick=&quot;alert('hi')
```

그러므로 `title` 속성에 새로운 임의의 HTML이 삽입되는 것을 방지할 수 있습니다. 이 이스케이핑은 `setAtttribute`처럼 기본 브라우저 API를 사용하므로 취약점은 오로지 브라우저 자체가 취약한 경우에만 존재할 수 있습니다.

## 잠재적 위험

어떠한 웹 어플리케이션에서도 정제되지 않은 사용자가 제공한 컨텐츠를 HTML, CSS 또는 자바스크립트로 실행하는 것은 잠재적으로 위험하므로 가능한 한 피해야 합니다. 그러나 일부 위험이 허용되는 경우가 있습니다.

예를 들어, CodePen 및 JSFiddle과 같은 서비스는 사용자가 제공한 컨텐츠를 실행합니다. 그러나 iframe 내부의 예상 가능하고 샌드박스(보호된 영역) 처리된 범위 내에서 실행될 것입니다. 이 경우 중요한 기능에 본질적으로 어느 정도의 취약점이 요구되는 경우, 취약점이 가능한 최악의 시나리오와 비교하여 기능의 중요성을 평가하는 것은 전적으로 여러분 팀의 책임입니다.

### HTML 주입

앞에서 배운대로, Vue는 자동으로 HTML 컨텐츠를 이스케이프하여 여러분의 어플리케이션에 실행 가능한 HTML이 실수로 주입되는 것을 방지합니다. 그러나 HTML이 안전하다는 것을 알고 있는 경우 HTML 내용을 명시적으로 렌더링 할 수 있습니다.

- 템플릿을 사용하는 경우:
  ```html
  <div v-html="userProvidedHtml"></div>
  ```

- 렌더 함수를 사용하는 경우:
  ```js
  h('div', {
    domProps: {
      innerHTML: this.userProvidedHtml
    }
  })
  ```

- JSX 렌더 함수를 사용하는 경우:
  ```jsx
  <div domPropsInnerHTML={this.userProvidedHtml}></div>
  ```

<p class="tip">샌드박스 iframe 또는 HTML을 작성한 사용자에게만 노출하는 앱의 일부에서 사용자가 작성한 HTML이 100% 안전하다고 간주할 수 없습니다. 또한 사용자가 자신의 Vue 템플릿을 작성할 수 있도록 하면 비슷한 위험이 따릅니다.</p>

### URL 주입

이와 같은 URL에서:

```html
<a v-bind:href="userProvidedUrl">
  click me
</a>
```

URL에 `javascript:`를 사용하여 JavaScript 실행을 막기 위한 "정제 작업"을 하지 않으면 잠재적 보안 문제가 있습니다. 이를 돕기 위해 [sanitize-url](https://www.npmjs.com/package/@braintree/sanitize-url) 과 같은 라이브러리가 있지만 다음을 참고하십시오.

<p class="tip">프론트엔드에서 URL 정제를 수행하는 중이라면 이미 보안 문제가 있는 것입니다. 사용자가 제공하는 URL은 항상 데이터베이스에 저장되기 전에 백엔드에서 정제되어야 합니다. 그러면 네이티브 모바일 앱을 비롯하여 API에 연결하는 _모든_ 클라이언트에 대해 그 문제가 발생하지 않습니다. 또한 정제된 URL을 사용하더라도, Vue는 안전한 목적지에 도달하는 것을 보장할 수 없습니다.</p>

### Styles 주입

이 예제를 보세요

```html
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="userProvidedStyles"
>
  click me
</a>
```

`sanitizedUrl`을 정제된 것으로 간주하여 자바스크립트가 아닌 실제 URL이라고 생각해 봅시다. 그러나 악의적인 사용자는 여전히 `userProvidedStyles`에 "클릭 잭" CSS를 제공할 수 있습니다. (예 : "로그인" 버튼 위의 투명한 상자에 링크 스타일 지정) 만약 `https://user-controlled-website.com/`이 어플리케이션의 로그인 페이지와 유사하게 빌드 된 경우 사용자의 실제 로그인 정보를 캡쳐했을 수 있습니다.

`<style>`요소에 사용자 제공 컨텐츠를 허용하는 것은 사용자에게 전체 페이지 스타일 지정을 완전히 제어 가능하도록 하므로 얼마나 더 큰 취약점을 발생시킬지 상상할 수 있습니다. 그렇기 때문에 다음과 같이 템플릿 내에서 스타일 태그가 렌더링되지 않습니다.

```html
<style>{{ userProvidedStyles }}</style>
```

클릭 재킹으로부터 사용자를 완전히 보호하려면 오직 샌드박스 iframe 내에서 CSS를 완전히 제어하도록 하는 것이 좋습니다. 또는 스타일 바인딩을 통해 사용자 제어를 제공할 때에는 [객체 구문](class-and-style.html#Object-Syntax-1)을 사용하여 다음과 같이 제어하기에 안전한 특정 속성에 대한 값만 제공하도록 허용하는 것이 좋습니다.

```html
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="{
    color: userProvidedColor,
    background: userProvidedBackground
  }"
>
  click me
</a>
```

### 자바스크립트 주입

템플릿과 렌더 함수에 부작용이 없어야 하므로 `<script>`요소를 Vue와 함께 렌더링하는 것은 옳지 않습니다. 그러나 이것이 런타임에 JavaScript로 평가되는 문자열을 포함시키는 유일한 방법은 아닙니다.

모든 HTML 요소는 `onclick`, `onfocus`, 그리고 `onmouseenter`와 같은 속성들의 값으로 자바스크립트 문자열을 받습니다. 사용자에게서 제공된 자바스크립트를 이러한 이벤트 속성에 바인딩하는 것은 잠재적 보안 위험이 있으므로 피해야 합니다.

<p class="tip">사용자가 제공한 JavaScript는 샌드박스 처리된 iframe 또는 오직 해당 JavaScript를 작성한 사용자에게만 노출될 수 있는 앱의 일부가 아니면 100% 안전하다고 간주할 수 없습니다.</p>

때때로 Vue 템플릿에서 크로스 사이트 스크립팅(XSS:cross-site scripting)을 수행하는 방법에 대한 취약성 보고서를 받습니다. 일반적으로 XSS를 허용하는 다음 두 가지 시나리오로부터 개발자를 보호할 실제적인 방법이 없기 때문에 이러한 경우를 실제 취약점으로 간주하지 않습니다.

1. 개발자가 명시적으로 사용자가 제공한 정제되지 않은 컨텐츠를 Vue 템플릿으로 렌더링하도록 요청하고 있습니다. 이것은 본질적으로 안전하지 않으며 Vue가 출처를 알 수 있는 방법이 없습니다.

2. 개발자가 서버 렌더링 및 사용자가 제공한 컨텐츠를 포함하는 전체 HTML 페이지를 Vue에 마운트 하고 있습니다. 이것은 근본적으로 \#1과 같은 문제이지만, 때때로 개발자들은 깨닫지 못합니다. 이로 인해 공격자가 일반 HTML로서는 안전하지만 Vue 템플릿으로서는 안전하지 않은 HTML을 제공하는 취약점이 발생할 수 있습니다. 가장 좋은 방법은 서버 렌더링 및 사용자 제공 컨텐츠를 포함할 수 있는 노드에 Vue를 마운트하지 않는 것입니다.

## 모범 사례

일반적으로 사용자가 제공한 정제되지 않은 컨텐츠를 HTML, JavaScript 또는 CSS로 실행되도록 허용한다면 여러분은 스스로를 공격에 노출시키는 것입니다. 이 조언은 실제로 Vue, 다른 프레임워크, 또는 심지어 프레임워크를 사용하지 않더라도 적용됩니다.

[잠재적 위험](#Potential-Dangers)에 대해 위에서 제시한 사항 외에도 다음 리소스를 숙지하는 것이 좋습니다.

- [HTML5 보안 치트 시트](https://html5sec.org/)
- [OWASP's Cross Site Scripting (XSS) 예방 치트 시트](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet)

그 다음 배운 내용을 사용하여 잠재적으로 위험한 패턴에 대해 종속성을 가진 소스 코드가 있는지, 그것들 중 써드파티 컴포넌트를 포함하고 있거나 DOM으로 렌더링되는 내용에 영향을 주는 경우가 있는지 검토하십시오.

## 백엔드 조정

CSRF/XSRF (cross-site request forgery) 및 XSSI (cross-site script inclusion)와 같은 HTTP 보안 취약점은 주로 백엔드에서 해결되므로 Vue의 문제는 아닙니다. 그러나 예를 들어 form 제출과 함께 CSRF 토큰을 제출하는 것과 같이 API와 상호작용하는 최고의 방법을 배우기 위해서는 백엔드 팀과 의사소통하는 것이 가장 좋습니다.

## 서버사이드 렌더링 (SSR)

SSR 사용시 몇 가지 추가 보안 문제가 있으므로 [SSR 설명서](https://ssr.vuejs.org/) 전체에 요약된 모범 사례를 따라 취약점을 피하십시오.
