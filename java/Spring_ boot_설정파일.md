# Spring boot 설정파일

스프링 부트는 외부 설정을 통해 스프링 부트 어플리케이션의 환경설정 혹은 설정값을 정할 수 있습니다. 
스프링 부트에서 사용할 수 있는 외부 설정은 크게 properties, YAML, 환경변수, 커맨드 라인 인수등이 있다.

이번 예제에는 프로젝트내의 application.properties에서 값 가져오는 방법을 정리하려고 한다.



@ConfigurationProperties 어노테이션을 사용하려면 META 정보를 생성해주는 spring-boot-configuration-processor 의존성이 필요하다

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```



application.properties 파일

```java
///myspringboot/src/main/resources/application.properties
hoon.name=Seung Hoon
hoon.age=${random.int(1,100)}
hoon.fullName=Jeong ${hoon.name}
```



### 1번째 방법

```java
@Component
public class MyRunner implements ApplicationRunner{
	@Value("${hoon.name}")
	private String name;
	
	@Value("${hoon.age}")
	private int age;
	
	@Override
	public void run(ApplicationArguments args) throws Exception{
		System.out.println("****************");
		System.out.println(name);
		System.out.println(age);
	}
}
```



### 2번째 방법

```java
@Component
@ConfigurationProperties("hoon")
public class HoonProperties {
	private String name;
	private int age;
	private String fullName; 
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public String getFullName() {
		return fullName;
	}
	public void setFullName(String fullName) {
		this.fullName = fullName;
	}
	
}
```



```java
@Component
@Order(1) // runner가 여러개일경우 우선순위가 적은 순으로 동작
public class MyRunner implements ApplicationRunner{	
	@Autowired
	HoonProperties hoonProperties;
	
	@Override
	public void run(ApplicationArguments args) throws Exception{
		System.out.println("****************");
		System.out.println(hoonProperties.getName());
		System.out.println(hoonProperties.getAge()+100);
		System.out.println(hoonProperties.getFullName());
	}
}
```



### 환경변수 우선순위

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties

2. 테스트에 있는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨드 라인 아규먼트
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로퍼티)에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수

11. RandomValuePropertySource

12. JAR 밖에 있는 특정 프로파일용 application properties
13. JAR 안에 있는 특정 프로파일용 application properties
14. JAR 밖에 있는 application properties
15. **JAR 안에 있는 application properties (현재 테스트 환경)**
16. @PropertySource
17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)