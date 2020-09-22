# Java 직렬화

* 자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)

- 시스템적으로 이야기하자면 JVM(Java Virtual Machine 이하 JVM)의 메모리에 상주(힙 또는 스택)되어 있는 객체 데이터를 바이트 형태로 변환하는 기술과 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 같이 이야기합니다.



### 예제 코드

직렬화에 사용된 Member클래스 정의

> 자바 기본(primitive) 타입과 `java.io.Serializable` 인터페이스를 상속받은 객체는 직렬화 할 수 있는 기본 조건을 가진다.

```java
public class Member implements Serializable {
    private String name;
    private String email;
    private int age;

    public Member(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }

    // Getter 생략

    @Override
    public String toString() {
        return String.format("Member{name='%s', email='%s', age='%s'}", name, email, age);
    }
}
```

#### 

#### Java 직렬화

```java
// Member객체 직렬화
Member member = new Member("local", "local@host.com", 27);
byte[] serializedMember;
try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
    try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
        oos.writeObject(member);
        // serializedMember -> 직렬화된 member 객체 
        serializedMember = baos.toByteArray();
    }
}
// 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
System.out.println(Base64.getEncoder().encodeToString(serializedMember));
//rO0ABXNyACBjb20ucmVzdC5hcGkudG9kb3MuZG9tYWluLk1lbWJlcmIFAo+9irfXAgADSQADYWdlTAAFZW1haWx0ABJMamF2YS9sYW5nL1N0cmluZztMAARuYW1lcQB+AAF4cAAAABt0AA5sb2NhbEBob3N0LmNvbXQABWxvY2Fs


// Member객체로 역직렬화
String base64Member = Base64.getEncoder().encodeToString(serializedMember);
byte[] deSerializedMember = Base64.getDecoder().decode(base64Member);
try (ByteArrayInputStream bais = new ByteArrayInputStream(deSerializedMember)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
        // 역직렬화된 Member 객체를 읽어온다.
        Object objectMember = ois.readObject();
        Member newMember = (Member) objectMember;
    }
}

System.out.println(newMember);
//Member{name='local', email='local@host.com', age='27'}

```



#### Json 직렬화

Gson 사용

```java
// Gson사용
Gson gson = new Gson();
// Member객체를 json으로 직렬화
Member local = new Member("local", "local@host.com", 27);
String result = gson.toJson(local);
System.out.println(result);
//{"name":"local","email":"local@host.com","age":27}

// json을 Member객체로 역직렬화
Member newLocal = gson.fromJson(result, Member.class);
System.out.println(newLocal);
//Member{name='local', email='local@host.com', age='27'}

```



#### 참고글

https://okky.kr/article/224715

https://woowabros.github.io/experience/2017/10/17/java-serialize.html