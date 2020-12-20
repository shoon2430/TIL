# CORS 이슈 해결하기(2)

리액트에서 어려개의 서버로 요청을 보내야 하는 경우가 있다. 이 경우 각 서버마다 cors문제가 발생하여 각각 proxy를 설정해 주어야 한다.



#### 이전처럼 package.json에 설정하는 것이 아니라 라이브러리를 이용하도록 한다.

```sh
npm install http-proxy-middleware
```

> 설치가 완료되면 적용할 프로젝트의 상단에 **setupProxy.js**를 만들어 작성한다.



#### 폴더 구조

* :file_folder: src
  * :file_folder: project
    * :file_folder: drectory1
    * :file_folder: drectory2
    * :memo: **setupProxy.js**

```javascript
# /src/project/setupProxy.js

const { createProxyMiddleware } = require("http-proxy-middleware");
module.exports = function (app) {

  //Service 1
  app.use(
    "/api/auth",
    createProxyMiddleware({
      target: "http://localhost:9001",
      changeOrigin: true,
    })
  );

  //Service 2
  app.use(
    "/api/accounts",
    createProxyMiddleware({
      target: "http://localhost:9002",
      changeOrigin: true,
    })
  );
  
  //Service 2
  app.use(
    "/api/todos",
    createProxyMiddleware({
      target: "http://localhost:9003",
      changeOrigin: true,
    })
  );

};
```

