---
layout: post
title: JS Observer Pattern
date: 2020-08-26 09:20:23 +0900
category: JS
---

# js 옵저버 패턴

> 바닐라 자바스크립트로 라이브러리없이 상태관리 시스템을 구현해보면서 옵저버패턴에 대해서 확실하게 알고 넘어가고자 정리를 하려고 한다.

### 옵저버 패턴이란

**옵저버 패턴**은 [객체](<https://ko.wikipedia.org/wiki/객체_(컴퓨터_과학)>)의 상태 변화를 관찰하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 [메서드](https://ko.wikipedia.org/wiki/메서드) 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 [디자인 패턴](https://ko.wikipedia.org/wiki/디자인_패턴)이다. 주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. [발행/구독 모델](https://ko.wikipedia.org/wiki/발행/구독_모델)로 알려져 있기도 하다.

###### 출처 [위키백과 옵저버 패턴](https://ko.wikipedia.org/wiki/옵서버_패턴)

## 구현

이 패턴의 핵심은 옵저버 또는 리스너(listener)라 불리는 하나 이상의 객체를 관찰 대상이 되는 객체에 등록시킨다.

그리고 각각의 옵저버들은 관찰 대상인 객체가 발생시키는 이벤트를 받아 처리한다.

![디자인패턴:옵저버패턴(Observer Pattern) – friday's blog](https://lh3.googleusercontent.com/proxy/MQ7Hpe9WD7n0Z1xZwrw33qOofVRkCeilGtO72BRDVHEwi5lxW9QTTZHG2IfUe4vMynIz-0WbgwPnl3pYevA3Q80HYUVKWgG6OzlHcLlb2escHITHsGk39vgCvFOJzA)

위 그림에서와 같이 Subject에서 이벤트가 발생하면 가각의 옵저버들은 관찰 대상인 객체가 발생시키는 이벤트를 받아 처리할 수 있다.

```javascript
class Observer {
  constructor() {
    this.handlers = [];
  }

  // 옵저버로 등록
  // @ eventName : 이벤트 이름
  // @ handler   : 해당 이벤트 발생시 실행 할 함수
  // @ context   : 옵저버로 등록 할 객체
  subscribe = (eventName, handler, context) => {
    let handlerArray = this.handlers[eventName];

    if (undefined === handlerArray) {
      this.handlers[eventName] = [];
      handlerArray = this.handlers[eventName];
    }

    handlerArray.push({
      handler: handler,
      context: context,
    });
  };

  // 이벤트 발생
  // 이벤트 발생시 옵저버로 등록되어 있는 객체들이 이벤트를 받아 처리가 가능하다
  // @ eventName : 이벤트 이름
  // @ data      : 사용되는 데이터
  publish = (eventName, data) => {
    let handlerArray = this.handlers[eventName];

    if (handlerArray === undefined) {
      console.log("handlerArray가 없을 경우 종료한다.");
      return;
    }

    for (var i = 0; i < handlerArray.length; i++) {
      const currentHandler = handlerArray[i];
      /* currentHandler 는 객체
        {
            handler: handler,
            context: context,
        }
      */
      currentHandler.handler(currentHandler.context, data);
    }
  };
}
```

위의 소스코드는 옵저버 클래스를 만들고 해당 옵저버에 등록할 수 있도록 하는 **subscribe**메서드와 옵저버에서 이벤트를 발생시키는 **publish**메서드를 만들었다.

```javascript
// 옵저버 객체 생성
const observer = new Observer();

const greetings = (context, data) => {
  console.log(`${context} ${data} `);
};

// 객체 생성
const orange = { greetings: greetings("안녕하세요", "orange") };
const apple = { greetings: greetings("안녕하세요", "apple") };
const pineapple = { greetings: greetings("안녕하세요", "pineapple") };
const watermelon = { greetings: greetings("안녕하세요", "watermelon") };

// 옵저버에 등록
observer.subscribe("hi", greetings, "orange");
observer.subscribe("hi", greetings, "apple");
observer.subscribe("hi", greetings, "pineapple");

orange.greetings;
apple.greetings;
pineapple.greetings;
watermelon.greetings;
observer.publish("hi", "반갑습니다.");

// < 실행결과 >
// 안녕하세요 orange
// 안녕하세요 apple
// 안녕하세요 pineapple
// 안녕하세요 watermelon

// orange 반갑습니다.
// apple 반갑습니다.
// pineapple 반갑습니다.
```

**publish**메서드를 실행함으로써 옵저버에 등록된 객체들에는 이벤트가 발생했지만, 옵저버에 등록 되지 않은 **watermelon**에 대해서는 이벤트가 발생하지 않은 것을 알 수 있다.
