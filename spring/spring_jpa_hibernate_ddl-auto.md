# spring.jpa.hibernate.ddl-auto

spring.jpa.hibernate.ddl-auto= create|create-drop|update|validate

- **create**
  JPA가 DB와 상호작용할 때 기존에 있던 스키마(테이블)을 삭제하고 새로 만드는 것을 뜻한다.
- **create-drop **
  JPA 종료 시점에 기존에 있었던 테이블을 삭제한다.
- **update**
  기존 스키마는 유지하고, 새로운 것만 추가하고, 기존의 데이터도 유지한다. 변경된 부분만 반영함
  주로 개발 할 때 적합하다.
- **validate**
  엔티티와 테이블이 정상 매핑 되어 있는지를 검증한다.

#### ex) validate인 경우

기존의 Entity클래스를 변경할 경우

```java
public class Account {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(unique = true)
	private String username;
	@Column
	private String password;
    //email 추가!
	private String email;

	@Builder
	public Account(String username, String password) {
		this.username = username;
		this.password = password;
	}
}
```

validate인 경우에는 검증만하기때문에 에러가난다.

![에러1](/images/spring/에러1.png)

![에러2](/images/spring/에러2.png)

해결방법은 update로 변경한다.

![test결과](/images/spring/test결과.png)

update로 변경 후에는 테이블에 email필드가 생기고 null값이 들거같것을 확인할 수 있다.
