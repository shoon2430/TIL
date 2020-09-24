# javascript 객체

### 객체 (Object)

JavaScript는 객체기반의 스크립트 언어이며 JavaScript를 이루고 있는 거의 모든 것은 객체이다. 객체란 **여러 속성을 하나의 변수에 저장할 수 있도록 해주는 데이터 타입**으로 Key / Value Pair를 저장할 수 있는 구조이다.

객체생성방법에는 **리터럴 방법**과 **생성자를 이용한 방법**이있다.

**리터럴 방법**

```javascript
const obj = {
  type1: "hoon", //문자열
  type2: 2020, //숫자
  type3: true, //boolean
  type4: {}, //객체 리터럴
  type5: [], //배열
  type6: function () {}, //함수
};
```

**생성자 함수를 이용한 방법**

```javascript
const obj = new Object();

obj.type1 = "string";
obj.type2 = 2020;
obj.type3 = true;
obj.type4 = {};
obj.type5 = [];
obj.type6 = function () {};
```

### 프로토타입(prototype)

JavaScript는 흔히 **프로토타입 기반 언어(prototype-based language)**라 불립니다.— 모든 객체들이 메소드와 속성들을 상속 받기 위한 템플릿으로써 **프로토타입 객체(prototype object)**를 가진다는 의미이다. 프로토타입 객체도 또 다시 상위 프로토타입 객체로부터 메소드와 속성을 상속 받을 수도 있고 그 상위 프로토타입 객체도 마찬가지이다.

이를 **프로토타입 체인(prototype chain)**이라 부르며 다른 객체에 정의된 메소드와 속성을 한 객체에서 사용할 수 있도록 하는 근간이다.

생성자 함수를 정의 (function의 이름은 대문자로 시작하는걸 권장한다.) 함수를 정의하면 함수만 생성되는 것이 아니라 Prototype Object도 같이 생성된다.

```javaScript
function Person(name, job) {
  this.name = name;
  this.job = job;
};
```

![prototype](/images/javascript/prototype.png)생성된 함수는 prototype이라는 속성을 통해 Prototype Object에 접근할 수 있습니다. Prototype Object는 일반적인 객체와 같으며 기본적인 속성으로 **constructor**와 \***\*proto\*\***를 가지고 있다.

![프로토1](/images/javascript/프로토1.png)

**constructor**는 Prototype Object와 같이 생성되었던 함수를 가리키고 있다. \***\*proto\*\***는 Prototype Link이다.

![프로토2](/images/javascript/프로토2.png)

예제를 한번 보자

```javascript
function Person(name, job) {
  this.name = name;
  this.job = job;
}

Person.prototype.age = 27;
Person.prototype.gender = "m";

const kim = new Person("kim", "job1");
const park = new Person("park", "job2");

console.log(kim.age); // 2
console.log(park.age); // 2
```

Prototype Object는 일반적인 객체이므로 속성을 마음대로 추가/삭제 할 수 있습니다. kim과 park은 Person 함수를 통해 생성되었으니 Person.prototype을 참조할 수 있게 된다.

![프로토4](/images/javascript/프로토4.png)

### Prototype Link

![프로토5](/images/javascript/프로토5.png)

kim에는 age 라는 속성이 없는데도 kim.age 를 실행하면 27라는 값을 참조하는 것을 볼 수 있습니다. 위에서 설명했듯이 Prototype Object에 존재하는 age 속성을 참조한다. 어떻게 age라는 값을 참조할 수 있는걸까? 바로 kim이 가지고 있는 \***\*proto\*\***가 그것을 가능하게 해주는 열쇠이다.

**prototype** 속성은 함수만**( Person.prototype )** 가지고 있던 것과는 달리 \***\*proto\*\***속성은 모든 객체가 빠짐없이 가지고 있는 속성이다.

\***\*proto**는 객체가 생성될 때 조상이었던 함수의 Prototype Object를 가리킵니다.\*\* kim객체는 Person함수로부터 생성되었으니 Person 함수의 Prototype Object를 가리키고 있는 것이다.

![prototype2](/images/javascript/prototype2.png)

![프로토6](/images/javascript/프로토6.png)

kim객체가 age 를 직접 가지고 있지 않기 때문에 age 속성을 찾을 때 까지 상위 프로토타입을 탐색합니다. 최상위인 Object의 Prototype Object까지 도달했는데도 못찾았을 경우 undefined를 리턴합니다. 이렇게 **proto**속성을 통해 상위 프로토타입과 연결되어있는 형태를 **프로토타입 체인(Chain)**이라고 한다.

이런 프로토타입 체인 구조 때문에 모든 객체는 Object의 자식이라고 불리고, Object Prototype Object에 있는 모든 속성을 사용할 수 있다. 한 가지 예를 들면 toString함수가 있다.
