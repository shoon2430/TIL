# React + Spring boot(4)

이번시간에는 지금까지 만들어온 Book 프론트 페이지와 Book Api서버를 연동해보도록 하겠습니다.

## 1. API를 이용하기 위한 폴더구조

기존의 폴더구조에서 api폴더를 추가합니다.

- :file_folder: book
  - :file_folder: api
    - 📄 : BookApi.js
    - :file_folder: model
      - 📄 : BookApiModel.js

> **BookApi.js** : Book Api서버로 요청을 하는 부분 입니다.
>
> **BookApiModel.js** : Book Api서버에서 Book에 대한 정보를 받을경우 데이터를 담아주기 위해 추가합니다.

## 2. BookApi 작성

Book Api로의 요청만 따로 관리하는 BookApi클래스를 작성합니다.

##### book/api/BookApi.js

```javascript
import axios from "axios";

class BookApi {
  // BookApi Url 리스트
  // GET    /api/books/
  // GET    /api/books/{ISBN}/
  // POST   /api/books/
  // PUT    /api/books/
  // DELETE /api/books/{ISBN}/

  //공통 적으로 사용되는 URL
  URL = "/api/books/";

  bookList() {
    return axios
      .get(this.URL)
      .then((response) => (response && response.data) || null);
  }

  bookDetail(ISBN) {
    return axios
      .get(this.URL + `${ISBN}/`)
      .then((response) => (response && response.data) || null);
  }

  bookCreate(bookApiModel) {
    return axios
      .post(this.URL, bookApiModel)
      .then((response) => (response && response.data) || null);
  }

  bookModify(bookApiModel) {
    return axios
      .put(this.URL, bookApiModel)
      .then((response) => (response && response.data) || null);
  }

  bookDelete(ISBN) {
    return axios
      .delete(this.URL + `${ISBN}`)
      .then((response) => (response && response.data) || null);
  }
}

export default BookApi;
```

> **axios** : Api로 비동기 통신을 하기 위해서 우리는 axios 라이브러리를 사용합니다.
>
> 설치가 안됬다면 `yarn add axios`

이전에 만들어둔 API서버와 요청할 수있도록 하는 함수들을 구현하였습니다.

## 3. 요청 받은 데이터를 담을 Model생성

book객체를 Json타입으로 받았을경우 받은 데이터를 담아둘 클래스를 작성합니다.

##### book/api/model/BookApiModel.js

```javascript
export default class BookApiModel {
  constructor(book) {
    this.isbn = book.isbn;
    this.title = book.title;
    this.author = book.author;
    this.publisher = book.publisher;
    this.price = book.price;
    this.imgUrl = book.imgUrl;
    this.introduce = book.introduce;
  }
}
```

> BookApi프로젝트에서 Book클래스에 해당하는 항목들을 동일하게 작성하였습니다.

## 4. BookStore에서 Api요청을 보낼 수 있도록 연결

이제 BookStore에서 임시데이터가 아닌 Api서버에서 데이터를 가지고 올 수 있도록 BookApi를 연결하고 사용해보도록 하겠습니다.

##### book/store/BookStore.js

```javascript
import { observable, computed, action } from "mobx";
//import booksData from "../data/Books"; 이제 사용하지않음
import BookApi from "../api/BookApi";
import BookApiModel from "../api/model/BookApiModel";

class BookStore {
  constructor() {
    this.bookApi = new BookApi();
  }

  //@observable books = booksData;
  //@observable book = booksData[0];
  @observable books = [];
  @observable book = null;

  @computed get _book() {
    return this.book ? { ...this.book } : {};
  }

  @computed get _books() {
    return this.books ? this.books.slice() : [];
  }

  @computed get getErrorMessage() {
    return this.errorMessage ? this.errorMessage : "FAIL";
  }

  @action select = (book) => {
    //this.book = book;
    this.bookDetail(book.isbn);
  };

  @action
  async bookList() {
    const books = await this.bookApi.bookList();
    this.books = books.map((book) => {
      return new BookApiModel(book);
    });
  }

  @action
  async bookDetail(ISBN) {
    const book = await this.bookApi.bookDetail(ISBN);
    this.book = new BookApiModel(book);
  }
}
```

> **@observable books = [];** : book리스트를 저장하고 있으므로 기본값으로 변경합니다.
>
> **@observable book = null;** : 기본값으로 변경합니다.
>
> **async bookList()** : Book Api에서 BookList를 가지고 온 뒤, Store의 books에 넣어주는 함수 입니다.
>
> **async bookDetail(ISBN)** : Book Api에서 Book을 가지고 온 뒤, Store의 book에 넣어주는 함수 입니다.

1. constructor() 생성자를 이용하여 bookstore 객체가 생성되었을 때, `this.bookApi = new BookApi();`를 하여 BookApi객체를 생성하여 넣어주었습니다.

2. select 함수는 기존의 방식도 가능하지만, 우리가 만든 bookDetail함수를 사용하여 선택했을 때, 선택된 도서가 자세히 보일 수 있도록 하였습니다.
3. 추가된 bookList와 bookDetail는 **async**를 사용하여 비동기로 실행되도록 하였고, **await**를 사용하여 Api 호출 후 데이터를 받은 다음 Store의 book, books에 값을 넣어주도록 하였습니다.

## 5. BookList 불러오기

메인 화면으로 이동시 Api를 호출하여 Store의 books를 세팅할 수 있도록 수정하겠습니다.

##### book/container/BookListContainer.js

```javascript
import React, { Component } from "react";
import BookListView from "../view/BookListView";
import { observer, inject } from "mobx-react";

@inject("bookStore")
@observer
class BookListContainer extends Component {
  //componentDidMount 추가
  componentDidMount() {
    const { bookStore } = this.props;
    bookStore.bookList();
  }

  onSelect = (book) => {
    const { bookStore } = this.props;
    bookStore.select(book);
  };

  render() {
    const { bookStore } = this.props;
    return <BookListView books={bookStore._books} onSelect={this.onSelect} />;
  }
}

export default BookListContainer;
```

> 리액트 생명주기 함수 중 하나인 **componentDidMount**를 오버라이딩 하여 **render**함수가 실행 된 후에 bookStore의 bookList함수를 실행하여 Api를 이용하여 Store의 books가 세팅되도록 수정합니다.

## 6. CORS 문제 해결

React에서 BookApi서버로 요청하는 과정에서 cors문제로 인해 데이터를 가지고 오지 못하는 상황이 발생한다.

![CORS_Error](/images/project/react_spring/part4/CORS_Error.png)

#### **Cross-Origin Resource Sharing(CORS)란**

추가 HTTP 헤더를 사용하여 브라우저가 한 출처에서 실행중인 웹 애플리케이션에 선택된 액세스 권한을 부여하도록하는 메커니즘입니다. 다른 출처의 자원. **웹 응용 프로그램은 자체와 다른 출처 (도메인, 프로토콜 또는 포트)를 가진 리소스를 요청할 때 cross-origin HTTP 요청을 실행**합니다. 출처 MDN

#### 해결 방법

해당 문제는 Proxy역할을 해주는 중간 서버를 만들어서 문제를 해결할수있다는 점을 찾았고, 또한 Webpack에서 간단한 방법으로 Proxy 기능을 지원해주기 때문에 package.json 파일에 다음 항목만 추가해주면 간단하게 CORS 문제를 해결할 수 있다는 점을 알게되었습니다.

##### package.json

```javascript
...
"proxy": "http://localhost:9000",
...
```

> react프로젝트의 package.js에 우리가 요청할 서버의 주소를 적어주면 됩니다.

BookApi.js의 URL을 변경해주자

##### book/api/BookApi.js

```javascript
class BookApi {

  //공통 적으로 사용되는 URL
  //URL = "http://localhost:9000/api/books/";
  URL = "/api/books/";
...
```

> 위 두가지 설정은 한 뒤, 서버를 껏다가 다시 실행해 줍니다.
>
> 실제 데이터베이스의 Book 테이블에 값들이 들어가 있을 경우 화면에 조회되는 것을 확인할 수 있습니다.

지금까지 React에서 BookApi를 이용하여 데이터를 가지고 와서 화면에 보여주는 부분을 구현하였습니다.

다음에는 book을 등록하고, 수정하고, 삭제하는 기능을 구현해보도록 하겠습니다.
