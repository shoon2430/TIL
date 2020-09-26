# sReact + Spring boot(2)

지난번 환경세팅에 이어서 book을 관리하는 페이지구현을 시작해보자

우리가 만들게될 Book관리 페이지는 다음과 같다.

![최종화면](/images/project/react_spring/part2/최종화면.png)

### 1. 먼저 mobx를 사용하기 때문에 BookStore를 만들어 관리할 수 있도록 한다.

##### src/book/store/BookStore.js

```javascript
import { observable, computed, action } from "mobx";
import booksData from "../data/Books";

class BookStore {
  @observable books = booksData;
  @observable book = booksData[0];

  @computed get _book() {
    return this.book ? { ...this.book } : {};
  }

  @computed get _books() {
    return this.books ? this.books.slice() : [];
  }

  @action select = (book) => {
    this.book = book;
  };

  @action
  async bookList() {
    this.books = await this.bookApi.bookList();
  }
}

//BookStore 객체를 싱글톤으로 생성해서 보내준다.
export default new BookStore();
```

> **BookData** : API연결전 사용 할 임시 데이터 [( 임시데이터 링크 )](https://github.com/shoon2430/Book-Front/blob/master/src/book/data/Books.js)
>
> **books** : 전체 book 리스트
>
> **book** : 현재 선택한 book
>
> **@observable** : mobx를 이용하여 관찰되도록 함
>
> **@computed** : 연산된 값
>
> **@action** : 상태변화를 일으키는것

[Mobx관련 참고 블로그](https://velog.io/@velopert/begin-mobx)

### 2. Provider 로 프로젝트에 BookStore 적용

##### src/index.js

```javascript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

//시멘틱 UI 사용
import "semantic-ui-css/semantic.min.css";
// Store등록을 위한 Provider
import { Provider } from "mobx-react";
// 방금전 만든 bookStore객체
import bookStore from "./book/store/BookStore";

ReactDOM.render(
  // Provider에 props로 bookStore를 넣어줍니다.
  <Provider bookStore={bookStore}>
    <App />
  </Provider>,
  document.getElementById("root")
);

serviceWorker.unregister();
```

### 3. 등록되어있는 bookList를 조회할 수 있도록 추가

![mockup_1](/images/project/react_spring/part2/mockup_1.png)

왼쪽에는 등록되어있는 Book리스트를 보여주기 때문에, BookList에 대한 로직을 구현할 **BookListContainer**와 리스트자체를 보여주는 **BookListView**, 각 아이템을 나타내는 **BookItemView**를 구현한다.

##### src/book/container/BookListContainer.js

```javascript
import React, { Component } from "react";
import BookListView from "../view/BookListView";
import { observer, inject } from "mobx-react";

@inject("bookStore")
@observer
class BookListContainer extends Component {
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

> **@inject** 함수는 mobx-react 에 있는 함수로서, 컴포넌트에서 스토어에 접근할 수 있게 해줍니다. 정확히는, 스토어에 있는 값을 컴포넌트의 props 로 "주입"을 해줍니다.
>
> **@observer**를 이용하여 주입된 스토어의 값이 변하는지 확인 할 수 있게 해줍니다.
>
> **BookListContainer**에서는 book선택 시 우측화면에 해당정보를 자세하게 보여줄 수 있도록하는 **onSelect**함수를 구현하고, 현재 스토어에 등록되어있는 book데이터와 onSelect함수를 **BookListView** 에 props로 보내줍니다.

##### src/book/view/BookListView.js

```javascript
import React, { Component } from "react";
import BookItemView from "./BookItemView";

class BookListView extends Component {
  render() {
    const { books, onSelect } = this.props;
    const bookList = books.map((book) => {
      return <BookItemView book={book} onSelect={onSelect} />;
    });

    return bookList;
  }
}
export default BookListView;
```

> **BookListContainer**에서 받은 books를 이용하여 book데이터마다 **BookItemView**를 만들어줍니다.

##### src/book/view/BookItemView.js

```javascript
import React, { Component } from "react";
import { Item, Divider } from "semantic-ui-react";

class BookItemView extends Component {
  render() {
    const { book, onSelect } = this.props;
    return (
      <Item.Group>
        <Item
          onClick={() => {
            onSelect(book);
          }}
        >
          <Item.Image size="small" src={book.imgUrl} />
          <Item.Content>
            <Item.Header as="a">{book.title}</Item.Header>
            <Item.Meta>
              <span className="price">{book.price}</span>
            </Item.Meta>
            <Item.Description>
              <p>{book.author}</p>
            </Item.Description>
          </Item.Content>
        </Item>
        <Divider />
      </Item.Group>
    );
  }
}

export default BookItemView;
```

> **BookListView**에서 받은 book의 정보와 onSelect함수를 적용하여 하나의 bookItem을 만들어줍니다.

### 4. 선택한 book의 자세한 정보를 보여줄 수 있도록 추가

![mockup_2](/images/project/react_spring/part2/mockup_2.png)

오른쪽에는 선택된 Book의 자세한 정보를 보여주기 위해, Store에서 데이터를 불러오는 **BookDetailsContainer**와 데이터를 이용하여 보여질 화면인 **BookDetailsView**을 구현한다.

##### src/book/container/BookDetailsContainer.js

```javascript
import React, { Component } from "react";
import { observer, inject } from "mobx-react";
import BookDetailsView from "../view/BookDetailsView";

@inject("bookStore")
@observer
class BookDetailsContainer extends Component {
  render() {
    const { bookStore } = this.props;
    return <BookDetailsView book={bookStore._book} />;
  }
}
export default BookDetailsContainer;
```

> **BookDetailsContainer**는 선택되어져 있는 book정보를 Store에서 가지고와서 **BookDetailsView**에 props로 정보를 넘겨줍니다.

##### src/book/view/BookDetailsView.js

```javascript
import React, { Component } from "react";
import { Card, Image } from "semantic-ui-react";

class BookDetailsView extends Component {
  render() {
    const { book } = this.props;

    return (
      <Card>
        <Image src={book.imgUrl} wrapped ui={false} />
        <Card.Content>
          <Card.Header>{book.title}</Card.Header>
          <Card.Meta>
            <span className="date">
              {book.author} &nbsp; {book.publisher}
            </span>
            <br></br>
            <span className="date">{book.price}</span>
          </Card.Meta>
          <Card.Description>{book.introduce}</Card.Description>
        </Card.Content>
        <Card.Content extra></Card.Content>
      </Card>
    );
  }
}
export default BookDetailsView;
```

> **BookDetailsContainer**에서 받은 book정보를 이용하여 book의 자세한 정보를 보여주는 카드를 만들어 줍니다.

### 5. 마지막으로 우리가 만든 것들을 App.js에 넣어줍니다.

먼저 두개의 container를 한 화면에서 보여지도록 모아줍니다.

##### src/book/view/menu/BookMain.js

```javascript
import React from "react";
import { Grid, Container } from "semantic-ui-react";
import BookListContainer from "../../container/BookListContainer";
import BookDetailsContainer from "../../container/BookDetailsContainer";

function BookMain() {
  return (
    <Container>
      <Grid columns={2} divided>
        <Grid.Column>
          <BookListContainer />
        </Grid.Column>
        <Grid.Column>
          <BookDetailsContainer />
        </Grid.Column>
      </Grid>
    </Container>
  );
}

export default BookMain;
```

> 시멘틱UI를 이용하여 화면을 2개로 나눈뒤 왼쪽에는 **BookListContainer**를 오른쪽에는 **BookDetailsContainer**를 넣어줍니다.

##### src/App.js

```javascript
import React from "react";
import BookMain from "./book/view/menu/BookMain";

function App() {
  return <BookMain />;
}

export default App;
```

> 우리가 만든 BookMain를 App.js에 넣어줍니다.

현재까지 임시 데이터를 이용해서 등록되어있는 Book의 리스트를 보여주고
Book을 선택시 자세한 정보를 보여줄 수 있는 화면을 구현해보았습니다.

다음은 Spring boot와 연동하여 임시 데이터가아닌 실제 로컬 데이터베이스에서 데이터를 가지고 올 수 있도록 구현해봅시다.
