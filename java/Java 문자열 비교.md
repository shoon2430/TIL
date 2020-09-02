# Java 문자열 비교

### 1. equals() method 에 의한 문자열 비교.

```java
public class StringTest {
	public static void main(String[] args) {
		// equals
		String str1 = "hello";
		String str2 = "hello";
		String str3 = new String("hello");
		String str4 = str3;

        System.out.println(str1.equals(str2));	// true
		System.out.println(str2.equals(str3));	// true
		System.out.println(str3.equals(str4));	// true

    }
}
```

equals() method는 문자열의 원본 내용을 비교하는 것이다.

즉, 동일한 문자열 값을 비교한다.

### 2. == 연산자(Operator)에 의한 문자열 비교.

```java
public class StringTest {
	public static void main(String[] args) {
		// ==
		String str1 = "hello";
		String str2 = "hello";
		String str3 = new String("hello");
		String str4 = str3;

        System.out.println(str1 == str2);	// true
		System.out.println(str2 == str3);	// false
		System.out.println(str3 == str4);	// true
    }
}
```

== 는 실제 값이 아닌 참조를 비교한다.

### 3. String = "" 과 new String("") 의 차이

위의 예제를 보면 str1과 str2가 true가 나오는 것을 확인 할 수 있다.

그림으로 그려보면 다음과 같다.

![](/images/java/String.png)

Java에서 문자열은 String Pool로 관리된다. "hello"라는 두 개의 문자열 변수를 지정했지만 JVM Heap 메모리의 String Pool에는 "hello"라는 문자열 하나만 존재한다. 두 변수 str1, str2는 참조로 가르키게 된다.

new 키워드로 생성했을 경우에는 Heap영역에 객체를 생성하게 된다. 변수 str3은 Heap영역의 "hello"를 가르키게 된다.
