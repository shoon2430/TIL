# React + Spring boot (1)

수업때 만들어본 예제를 정리해 보려고한다. 환경설정부터 자세하게 기록해놓으면 나중에 프로젝트를 할 때 참고하면서 도움이 될 것 같아 개인적으로 라도 많은 도움이 될 것 같다.

예제로 만들어 볼 프로젝트는 React 와 Spring boot를 이용하여 기본적인 도서관리를 위한 book CRUD 웹페이지를 구현하려고한다.

react에서는 상태관리를 위해 mobx를 사용하고, 데이터를 저장하기위해서 DB는 mariaDB를 사용한다.

## 1. 리액트 프로젝트 생성

```bash
yarn create react-app book-front
```

## 2. 라이브러리 설치

사용할 라이브러리

1. UI디자인을위해 시멘틱 UI사용
   `semantic-ui-react semantic-ui-css`

2. react에서 State상태 관리를 위해 mobx사용

   `mobx --save mobx-react --save`

3. spring-boot서버와 통신하기 위해 axios사용

   `axios`

### 2-1. 라이브러리 설치

```bash
yarn add semantic-ui-react semantic-ui-css mobx mobx-react axios
```

### 2-2. mobx의 어노테이션을 사용하기위해 babel 플러그인을 추가해준다.

플러그인을 추가하기위해 yarn eject를 이용한다.

```
yarn eject
```

> **yarn eject** 를 하면 숨겨져 있던 웹팩, 바벨 등의 설정을 보여주고 이것을 커스터마이징 할 수 있도록 해주는 명령어. \*주의 : 한번 **eject**를 하면 이전 상태로 돌아갈 수 없습니다.

eject시 밑에와 같은 에러가 발생할 수 도있다.

![eject_error_1](/images/project/react_spring/part1/eject_error_1.png)

```
//해결 방법
git init
git add .
git commmit -m "first commmit"
//다시 eject
yarn eject
```

바벨 플러그인 추가

eject된 package.json의 **babel** 부분에서 플러그인을 추가한다.

```
//pakage.json

"babel": {
    "presets": [
      "react-app"
    ],
    "plugins": [
      [
        "@babel/plugin-proposal-decorators",
        {
          "legacy": true
        }
      ],
      [
        "@babel/plugin-proposal-class-properties",
        {
          "loose": true
        }
      ]
    ]
  }
```

> **@babel/plugin-proposal-decorators**와 **@babel/plugin-proposal-class-properties** 이부분이 추가

## 3. 프로젝트 전체 폴더 구조

[프로젝트 링크](https://github.com/shoon2430/Book-Front)

```
git clone https://github.com/shoon2430/Book-Front.git
```

- :file_folder: src

  - :page_facing_up: index.js

  - :page_facing_up: App.js

  - :file_folder: book

    - :file_folder: api
    - :file_folder: store
    - :file_folder: container
    - :file_folder: view
    - :file_folder: data​

### 폴더구조 설명

> :file_folder: **api** : Spring Boot 서버로 요청하는 부분
>
> :file_folder: **store** : mobx를 이용하여 book에 대한 상태관리를 하는 역활
>
> :file_folder: **container** : view에대한 로직등을 구현하여 props로 내려주는 역활
>
> :file_folder: **view** : props에 따라 데이터가 어떻게 보이는지, view로서의 역활
>
> :file_folder: **data** : Spring Boot로 데이터를 요청하기전 임시데이터를 사용하기 위함

그럼 다음은 폴더 구조대로 book을 관리할 수 있는 페이지를 구현해보자
