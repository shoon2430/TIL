# String vs StringBuffer/ StringBuilder

Java에서 문자열을 다루는 대표적인 클래스로 **String**, **StringBuffer**, **StringBuilder** 가 있다. 이전에 면접을 보면서 나왔던 질문이기 때문에 따로 정리해두려고 한다.

String과 StringBuffer/ StringBuilder 클래스의 가장 큰 차이점은 String은 불편(immutable)의 속성을 갖는다는 점이다.

```java
String str = "hello";		  // String Pool에 저장
// String str = new String(); // heap에 저장

str = str + "world" // hello world
```

위와 같은경우는 str이 가리키는 곳에 저장된 "hello"에 "world"를 더해 "hello world"로 변경된 것으로 보일 수있지만

![스트링vs스트링버퍼](/images/java/스트링vs스트링버퍼_1.png)

위와 같이 String은 불변성을 가지기 때문에 **변하지 않는 문자열을 자주 읽어들이는 경우** **`String`**을 사용하면 좋은 성능을 기대할 수 있다.

하지만 문자열 **추가, 수정, 삭제 등의 연산이 빈번하게 발생하는 경우**에 **`String`** 을 사용할 경우 **Heap**에 많은 임시 **Garbage**가 생성되어 힙메모리 부족으로 어플리케이션 성능에 문제가 될 수 있다.

이를 해결하기 위해 Java에서는 **가변(mutable)**성을 가지는 **StringBuffer/ StringBuilder**클래스가 있다.

**StringBuffer/ StringBuilder**는 **String**과는 반대로 가변성을 가지기 때문에 .append(), .delete() 등의 API를 이용하여 **동일 객체내에서 문자열을 변경**하는 것이 가능하다.

```java
StringBuffer sb = new StringBuffer("hello");
sb.append(" world")
```

![스트링vs스트링버퍼_2](/images/java/스트링vs스트링버퍼_2.png)

### StringBuffer vs StringBuilder

**StringBuffer**와 **StringBuilder**의 가장큰 차이점은 **동기화 유무**로써 **StringBuffer**는 동기화 키워드를 지원하여 **멀티쓰레드 환경에서 안전하다는점(thread-safe)**이다.

반대로 **StringBuilder**는 **동기화를 지원하지 않기**때문에 멀티쓰레드 환경에서 사용하는 것은 적합하지 않지만 동기화를 고려하지 않는 만큼 **단일쓰레드에서의 성능은 StringBuffer보다 뛰어나다.**

## 정리

- **String** : 문자열 연산이 적고 멀티쓰레드 환경일 경우
- **StringBuffer** : 문자열 연산이 많고 멀티쓰레드 환경일 경우
- **StringBuilder** : 문자열 연산이 많고 단일쓰레드이거나 동기화를 고려하지 않아도 되는 경우
