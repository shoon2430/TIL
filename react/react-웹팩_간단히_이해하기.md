# 웹팩 간단히 이해하기

리액트를 공부하면서 yarn start를 할 때 웹팩이 실행되는 것을 보고 웹팩이 어떠한 일들을 하는 것인지 알고 싶어 정리하게 되었다.

## 웹팩

![img](/images/react/웹팩메인.png)

**웹팩**은 오픈 소스 자바스크립트 **모듈 번들러**로써 여러개로 나누어져 있는 파일들을 하나의 자바스크립트 코드로 압축하고 최적화하는 라이브러리이다.

## 모듈

**모듈**이란 관련된 객체들의 집합소.

어떠한 기능을 수행하기 위해 함수 또는 객체들을 만들어 놨으면.

그걸 한 .js의 파일에 써놓기엔 가독성이나 유지보수가 좋지 않아서 관련 함수 또는 객체들을

.js파일별로 따로 모아놓은것들을 모듈이라고 생각하면 될거같다.

간단한 예를 들어보자

```javascript
<!-- index.html -->
<html>
  <head>
    <script src="./source/hello.js"></script>
    <script src="./source/world.js"></script>
  </head>
  <body>
    <h1>Hello, Webpack</h1>
    <div id="root"></div>
    <script>
      document.getElementById("root").innerHTML = word;
    </script>
  </body>
</html>
// hello.js
var word = "Hello";
// world.js
var word = "World";
```

해당코드를 웹브라우저에서 실행시켜보면 결과는 다음과 같다.

![img](/images/react/웹팩1.png)

실행결과는 각각의 스크립트 파일에 같은 변수를 사용하고 있으므로 밑에 있는 world.js의 word의 값이 추가되는 것을 확인할 수 있다.
만약 수백 개의 자바스크립트코드를 로드해서 쓰고 있고, 그리고 그것을 개발한 사람들이 모두 다르다고 가정했을 때에도 지금과 같이 변수 이름의 충돌이 있을 수도 있다.

이러한 방법을 해결하기위해서는 **모듈**이라는 방법을 사용한다.

소스코드를 수정해보자

```javascript
<!-- index.html -->
<html>
  <head>
    <!-- <script src="./source/hello.js"></script>
    <script src="./source/world.js"></script> -->
  </head>
  <body>
    <h1>Hello, Webpack</h1>
    <div id="root"></div>
    <script type="module">
      import hello_word from "./source/hello.js";
      import world_word from "./source/world.js";
      document.getElementById("root").innerHTML = hello_word + " " + world_word;
    </script>
  </body>
</html>
// hello.js
var word = "Hello";
export default word;
// world.js
var word = "World";
export default word;
```

해당코드를 웹브라우저에서 실행시켜보면 결과는 다음과 같다.

![img](/images/react/웹팩3.png)

네트워크 속도가 빠르다고 해서 웹 페이지가 완벽하게 로드되는 시간까지 빠른 것은 아니다.
예를 들어 웹 페이지를 구성하는 html, js, css 및 이미지 파일들이 수백 개가 넘어간다면 그 개수만큼 웹 서버에 요청하여 네트워크 커넥션이 많아질 것이고, 네트워크에 부하가 많이 걸리면서 속도가 늘어날 것이다.

여기서 동일한 타입의 파일끼리 묶어서 구성하면 웹 서버에 요청하는 수가 줄지 않을까??

## 번들러

이런 고민을 해결해 줄 수 있는 것이 번들러입다.

번들러는 지정한 단위로 파일들을 하나로 만들어서 요청에 대한 응답으로 전달할 수 있은 환경을 만들어주는 역할을 한다.

그럼 이제 번들러중 하나인 웹팩을 도입해보자.

웹팩으로 얻을수있는 장점은 리팩토링이다. 구동되는 코드는 그대로 유지하면서 내부의 코드는 더 효율적으로 바꿀수 있고, 구형브라우저에서도 사용가능할수있고, 우리가 원하는대로 번들링을 할수있다.

```javascript
// 웹팩 설치
npm init => pakage.json 생성
npm install -D webpack webpack-cli => 웹팩 설치
```

우선은 우리가 사용하고있는 hello.js와 world.js를 index.js파일안에 모아 entry파일을 만들어준다.

```javascript
// index.js
// entry 파일
import hello_word from "./hello.js";
import world_word from "./world.js";
document.getElementById("root").innerHTML = hello_word + " " + world_word;
```

웹팩 실행

index.js와 index.js에 사용하고있는 모든 파일들을 하나로 만든다.

```javascript
npx webpack --entry ./source/index.js --output ./public/index_bundle.js
```

index_bundle.js 파일이 만들어진것을 확인 할 수있다.

![img](/images/react/웹팩5.png)

이제 소스코드를 수정해보자

```javascript
<!-- index.html -->
<html>
  <head></head>
  <body>
    <h1>Hello, Webpack</h1>
    <div id="root"></div>
    <script src="public/index_bundle.js"></script>
  </body>
</html>
```

이전의 코드와 동일하게 실행되는것을 확인할 수있다.

![img](/images/react/웹팩2.png)기존(왼쪽) / 현재(오른쪽)

그리고 기존의 hello.js와 world.js가 index_bundle.js로 통합된것을 확인할수있다.
