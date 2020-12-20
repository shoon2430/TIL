# CORS 이슈 해결하기(1)

리액트에서 axios를 이용하여 네이버 밴드의 데이터를 가져오는 작업을 하는 도중 CORS라는 에러가 발생하여 데이터를 가지고 오지 못하는 문제에 직면했다. get 요청을 보낸 url을 주소창에 넣어보면 데이터는 정상적으로 가지고 오는 것을 확인할 수 있지만 실제 화면에서는 에러가 나고 있었다.

![img](/images/react/cors.png)에러 내용

#### **Cross-Origin Resource Sharing(CORS)란**

추가 HTTP 헤더를 사용하여 브라우저가 한 출처에서 실행중인 웹 애플리케이션에 선택된 액세스 권한을 부여하도록하는 메커니즘입니다. 다른 출처의 자원. **웹 응용 프로그램은 자체와 다른 출처 (도메인, 프로토콜 또는 포트)를 가진 리소스를 요청할 때 cross-origin HTTP 요청을 실행**합니다.
출처 MDN

#### **CORS문제 해결**

해당 문제는 Proxy역할을 해주는 중간 서버를 만들어서 문제를 해결할수있다는 점을 찾았고, 또한 Webpack에서 간단한 방법으로 Proxy 기능을 지원해주기 때문에 package.json 파일에 다음 항목만 추가해주면 간단하게 CORS 문제를 해결할 수 있다는 점을 알게되었다.

```javascript
// package.js
"proxy": "https://openapi.band.us",
```

package.js파일에 해당 proxy를 추가한뒤

```javascript
// getBandData.js

// axios.get('https://openapi.band.us/v2.1/bands?access_token=12345'); 수정전
axios.get("/v2.1/bands?access_token=12345"); // 수정후
```

실제 요청하는 url을 수정해주면 된다.

참고블로그

[https://velog.io/@jmkim87/%EC%A7%80%EA%B8%8B%EC%A7%80%EA%B8%8B%ED%95%9C-CORS-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90](https://velog.io/@jmkim87/지긋지긋한-CORS-파헤쳐보자)

https://snowdeer.github.io/openshift/2020/06/13/react-for-cors-using-proxy/
