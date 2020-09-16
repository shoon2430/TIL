# 스프링 부트 DB연동하기

## JPA 연동

#### 1\$ 의존성을 추가한다.

```java
//pom.xml
<!-- h2 memory DB 추가 -->
	<dependency>
    	<groupId>com.h2database</groupId>
    	<artifactId>h2</artifactId>
        <scope>runtime</scope>
	</dependency>
<!-- boot data JPA 추가 -->
	<dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
<!-- MySql connector 추가 -->
	<dependency>
    	<groupId>mysql</groupId>
    	<artifactId>mysql-connector-java</artifactId>
    	<version>8.0.13</version>
    </dependency>

```

테스트를위한 h2 memory DB와, JPA 그리고 mySql연결을위한 의존성을 추가한다.

#### 2\$ application.properties에 DB관련 설정을 추가한다.

```java
#/myJpaExampleProject/src/main/resources/application.properties
# tomcat 포트 수정
server.port=8088

# Hibernate 환경에서 DB 초기화
spring.jpa.hibernate.ddl-auto=update
# 콘솔에서 sql를 로깅 하기위함
spring.jpa.show-sql=true

# DB설정 설정
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/spring_db?useUnicode=true&charaterEncoding=utf-8&useSSL=false&serverTimezone=UTC
spring.datasource.username=spring
spring.datasource.password=spring
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver

# MySQL 상세 지정
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```

#### 3\$ DB연결 확인

```java
///myJpaExampleProject/src/main/java/com/example/jpa/myJpaExampleProject/runner/DBRunner.java
@Component
public class DBRunner implements ApplicationRunner{
	private final DataSource dataSource;

	DBRunner(DataSource dataSource){
		this.dataSource = dataSource;
	}

	private Logger logger = LoggerFactory.getLogger(DBRunner.class);

	@Override
	public void run(ApplicationArguments args) throws Exception {
		Connection connection = dataSource.getConnection();
		DatabaseMetaData dbMeta = connection.getMetaData();
		logger.info("DataSource 구현 클래스 이름 : "+dataSource.getClass().getName());
		logger.info("DB URL : "+dbMeta.getURL());
		logger.info("DB DriverName : "+dbMeta.getDriverName());
		logger.info("DB Username : "+dbMeta.getUserName());
	}
}
```

MySQl과 연결확인을 위해 Runner 클래스를 만들어 어플리케이션 실행 시점에 확인해본다.

![db연결확인](/images/spring/db연결확인.png)

서버 실행시 DB가 연결이 되었으면 DB정보가 출력되는것을 확인할 수 있다.

#### 4\$ 간단한 CRUD API 구현

```java
<!-- devtools 수정시 서버를 껏다키지 않아도됨!! -->
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-devtools</artifactId>
    	<optional>true</optional>
    </dependency>
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
    <dependency>
    	<groupId>org.projectlombok</groupId>
    	<artifactId>lombok</artifactId>
    	<version>1.18.12</version>
    	<scope>provided</scope>
    </dependency>
```

소스코드 수정시 서버를 restart해주는 **devtools**와 @Getter, @Setter등을 자동으로 생성해주어 소스코드 작성을 편리하게 하기위해 **lombok**을 설치한다.

#### 프로젝트 구조

- :file_folder:myJpaExampleProject

  - **MyJpaExampleProjectApplication.java**

- :file_folder:myJpaExampleProject/controller

  - **StudentController.java**

- :file_folder:myJpaExampleProject/entity

  - **Student.java**

- :file_folder:myJpaExampleProject/repository

  - **StudentRepository.java**

- :file_folder:myJpaExampleProject/runner

  - **DBRunner.java**

#### 4-1\$ entity

```java
// myJpaExampleProject/entity/Students.java
@Setter
@Getter
@NoArgsConstructor
@Entity
public class Student {
	@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@Column(unique = true) //name을 고유한값으로
	private String name;

	private int age;
	private String gender;
	private String grade;

	@Builder
	public Student(Long id, String name, int age, String gender, String grade) {
		super();
		this.id = id;
		this.name = name;
		this.age = age;
		this.gender = gender;
		this.grade = grade;
	}

	@Override
	public String toString() {
		return "Students [name=" + name + ", age=" + age + ", gender=" + gender + ", grade=" + grade + "]";
	}
}
```

> **@GeneratedValue(strategy = GenerationType.IDENTITY)**
>
> - 기본 키 생성을 데이터베이스에 위임
> - id 값을 null로 하면 DB가 알아서 AUTO_INCREMENT 해준다. Ex) MySQL, PostgreSQL, SQL Server DB2 등
>
> **@Column(unique = true)**
>
> - 해당 컬럼에 unique속성을 부여한다.

#### 4-2\$ repository

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
	Optional<Student> findByName(String name);
}
```

> **JpaRepository**를 상속받고, name으로 조회 할 수 있는 메서드를 추가한다.

#### 4-3\$ controller

```java
@RequiredArgsConstructor
@RestController
public class StudentController {

	private final StudentRepository repository;

	@GetMapping("/api/v1/students")
	public List<Student> getStudents() {
		return repository.findAll();
	}

	@PostMapping("/api/v1/students")
	public Student createStudent(@RequestBody Student student) {
		return repository.save(student);
	}

	@GetMapping("/api/v1/students/{id}")
	public Student getStudent(@PathVariable Long id) {
		return repository.save(repository.findById(id).get());
	}

	@PutMapping("/api/v1/students/{id}")
	public Student setStudent(@PathVariable Long id, @RequestBody Student studentDetail){
		Student student = repository.findById(id).get();
		student.setName(studentDetail.getName());
		student.setAge(studentDetail.getAge());
		student.setGender(studentDetail.getGender());
		student.setGrade(studentDetail.getGrade());
		return repository.save(student);
	}

	@DeleteMapping("/api/v1/students/{id}")
	public ResponseEntity<String> deleteStudent(@PathVariable Long id) {
		repository.delete(repository.findById(id).get());
		return new ResponseEntity<String>("succees",HttpStatus.OK);
	}
}
```

> **@RequiredArgsConstructor**
>
> - 초기화 되지않은 `final` 필드나, `@NonNull` 이 붙은 필드에 대해 생성자를 생성해 준다.
>
> **@RestController**
>
> - 간단히는 ( **@Controller** + **@ResponseBody **) return시 **Json 형태**로 객체 데이터를 반환
>
> **ResponseEntity**
>
> - ResponseEntity오브젝트는 HTTP 응답의 body에 삽입할 오브젝트를 소유한다.
> - ResponseEntity오브젝트는 body오브젝트에 추가해 statusCode와 HTTP응답 헤더를 설정할 수 있다.

#### 5\$ 결과

### POST /api/v1/students

<img src=/images/spring/spring_boot_jpa_post.png align="left">

### GET /api/v1/students

<img src=/images/spring/spring_boot_jpa_get.png align="left">

### GET /api/v1/students/{id}

<img src=/images/spring/spring_boot_jpa_get_detail.png align="left">

### PUT /api/v1/students/{id}

<img src=/images/spring/spring_boot_jpa_put.png align="left">

### DELTE /api/v1/students/{id}

<img src=C:\Users\jeongseunghoon\Desktop\delete.png align="left">
