# React + Spring boot(3)

지금까지 우리는 등록되어있는 book리스트를 조회하는 페이지를 구현했습니다. 이번에는 임시 데이터가 아닌 우리 실제 데이터베이스에 저장되어있는 데이터를 이용해서 조회하는 페이지를 구현해보려고 합니다.

[프로젝트 링크](https://github.com/shoon2430/Book-Api)

```
git clone https://github.com/shoon2430/Book-Api.git
```

## 1. Spring boot 프로젝트 생성

![프로젝트_생성_1](/images/project/react_spring/part3/프로젝트_생성_1.png)

![프로젝트_생성2](/images/project/react_spring/part3/프로젝트_생성2.png)

![프로젝트_생성_3](/images/project/react_spring/part3/프로젝트_생성_3.png)

## 2. dependency 추가

bookApi를 위한 라이브러리를 추가한다.

#### pom.xml

```xml
<!-- h2 DB -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- mariadb -->
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
</dependency>

<!-- JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!--gson -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>

<!-- devtools -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>

<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

> **h2 DB** : 테스트를 위한 DB추가
>
> **mariadb** : mariaDB와 연결하기위해 추가
>
> **JPA**: Spring JPA를 사용하기 위해 추가
>
> **gson** : 객체를 Json형태로 변환하기 위해 추가
>
> **devtools** : 코드 수정시 서버가 재실행되도록 하는 유틸리티 추가
>
> **lombok** : getter,setter등 보일러 코드들을 간편하게 작성하기위한 유틸리티 추가

## 3. Maria DB 연결

#### application.yml

```js
server:
  port: 9000

spring:
# Maria DB 설정
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:3306/spring_db?useUnicode=true&charaterEncoding=utf-8&useSSL=false&serverTimezone=UTC
    username: spring
    password: spring
  jpa:
    database-platform: org.hibernate.dialect.MariaDB53Dialect
    hibernate:
      ddl-auto: create
```

> **server** : 9000포트로 변경
>
> **datasource** : MariaDB에 맞게 드라이버, url, DB이름, username, password를 설정해준다.
>
> **jpa** : 데이터베이스 플랫폼을 MariaDB에 맞게 설정해주고, [**ddl_auto**](https://github.com/shoon2430/TIL/blob/master/spring/spring_jpa_hibernate_ddl-auto.md)를 설정해준다.

## 4. 프로젝트 폴더 구조

- :file_folder: books
  - :file_folder: application
  - :file_folder: domain
    - :file_folder: service​
      - :file_folder: logic
  - :file_folder: store
    - :file_folder: logic
    - :file_folder: ​ repository

> **application** : controller가 위치
>
> **domain** :
>
> **store** :

## 5. Book CRUD 구현

### 5-1. domian 정의

react에서 임시데이터로 사용한 데이터를 그대로 domian으로 등록하여 구현한다,

#### /books/domain/Book.java

```java
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
@Entity
public class Book {

	@Id
	private String ISBN;
	private String title;
	private String author;
	private String publisher;
	private double price;
	private String imgUrl;
	private String introduce;

	@Override
	public String toString() {
		Gson json = new Gson();
		return json.toJson(this);
	}

	public static Book sample(){
		 String ISBN = "9781617293986";
	     String title = "Spring Microservices in Action";
	     String author= "John Carnell";
	     String publisher= "Manning";
	     double price= 59.92;
	     String imgUrl= "book_images/spring.jpg";
	     String introduce= "Spring Boot and Spring Cloud offer Java developers an easy " +
	                    "migration path from traditional monolithic Spring applications to " +
	                    "microservice-based applications that can be deployed to multiple " +
	                    "cloud platforms.";

	     return new Book(ISBN,title,author,publisher,price,imgUrl,introduce);
	}
}
```

> **@Id** : ISBN를 ID로 지정 합니다.
>
> **toString** : **Gson**라이브러리를 사용하여 **json**형태로 리턴할 수 있도록 Override 합니다.
>
> **sample** : 테스트용으로 sample데이터 객체를 리턴하는 메서드 입니다.

### 5-2. Service 정의

우리가 필요한 기능들이 어떤 것 들이 있는지 정의 합니다.

#### books/domain/service/BookService.java

```java
import com.book.api.books.domain.Book;
import com.book.api.books.store.exceptions.DuplicateException;

public interface BookService {
	// 만들어야 할 것
	// book List   정보 => bookList()
	// book detail 정보 => bookDetail(String ISBN)
	// book create 기능 => bookCreate(Book book)
	// book modify 기능 => bookModify(Book book)
	// book delete 기능 => bookDelete(String ISBN)

	public abstract List<Book> bookList() throws NoSuchElementException;
	public abstract Book bookDetail(String ISBN) throws NoSuchElementException;
	public abstract void bookCreate(Book book) throws DuplicateException;
	public abstract void bookModify(Book book) throws NoSuchElementException;
	public abstract void bookDelete(String ISBN) throws NoSuchElementException;
}
```

> **만들어야 할 것** : 우리는 기본적인 Book CRUD를 구현하려고 합니다.
>
> **NoSuchElementException** : 우리가 요청한 데이터가 존재하지 않는경우 발생시키는 Exception 입니다.
>
> **DuplicateException** : Exception을 상속받아 만든 우리만의 Exception입니다. (지금은 작성하지 않아도 됩니다. 이후에 작성하도록 하겠습니다.)

### 5-3. Service 구현체 생성

위에서 우리가 정의한 service Interface를 구현하는 클래스 입니다.

#### books/domain/service/logic/BookServiceImpl.java

```java
@RequiredArgsConstructor
@Service
public class BookServiceImpl implements BookService{
// 만들어야하는 메서드를 실제 구현 하는 클래스

@Override
public List<Book> bookList() throws NoSuchElementException {
	// TODO Auto-generated method stub
	return null;
}

@Override
public Book bookDetail(String ISBN) throws NoSuchElementException {
	// TODO Auto-generated method stub
	return null;
}

@Override
public void bookCreate(Book book) throws DuplicateException {
	// TODO Auto-generated method stub
}

@Override
public void bookModify(Book book) throws NoSuchElementException {
	// TODO Auto-generated method stub
}

@Override
public void bookDelete(String ISBN) throws NoSuchElementException {
	// TODO Auto-generated method stub
}
}
```

> 현재는 데이터베이스쪽으로 요청을 보내는 기능이 구현이 되지않았기 때문에 파일만 만들어놓고 다음으로 넘어가도록 합시다.

### 5-4. BookStore 정의

JPA를 사용하여 요청하는 부분을 Service와 나누기 위해서 BookStore를 작성합니다.

#### books/store/BookStore.java

```java
public interface BookStore {

	// Book테이블에 Select 요청 => retrieveAll()
	// Book테이블에 ISBN 기준으로 Select 요청 => retrieve(String ISBN)
	// Book테이블에 Insert 요청 => create(Book book)
	// Book테이블에 Update 요청 => update(Book book)
	// Book테이블에 Delete 요청 => delete(String ISBN)
	// Book테이블에 ISBN으로 값을 찾을 때 값이 존재하는지 확인 => isExist(String ISBN)

	public abstract List<Book> retrieveAll() throws NoSuchElementException;
	public abstract Book retrieve(String ISBN) throws NoSuchElementException;
	public abstract void create(Book book) throws DuplicateException;
	public abstract void update(Book book) throws NoSuchElementException;
	public abstract void delete(String ISBN) throws NoSuchElementException;
	public abstract boolean isExist(String ISBN);
}

```

> service와 Store를 나누는 이유는 추후에 JPA가 아닌 Mybatis를 사용하게 될 경우 Service의 소스코드가 변경이 되어야 할 수도 있기때문에 확장성을 위해 나누었습니다.

### 5-5 BookJpo 구현

실제 데이터베이스의 테이블 정보와 연계되는 BookJpo클래스를 생성한다.

#### books/store/repository/BookJpo.java

```java
@AllArgsConstructor
@NoArgsConstructor
@Setter
@Getter
@Entity
@Table(name="BOOKS")
public class BookJpo {
	@Id
	//@GeneratedValue(strategy = GenerationType.IDENTITY)
	private String ISBN;
	private String title;
	private String author;
	private String publisher;
	private double price;
	private String imgUrl;
	private String introduce;


	public BookJpo(Book book) {
		BeanUtils.copyProperties(book, this);
	}

	//BookJpo를 Book으로 변환
	public Book toDomain() {
		Book book = new Book();
		BeanUtils.copyProperties(this, book);
		return book;
	}

	//BookJpo List를 Book List로 변환
	public static List<Book> toDomains(List<BookJpo> bookJpos){
		return bookJpos.stream().map(BookJpo::toDomain).collect(Collectors.toList());
	}
}

```

> **@Table(name="BOOKS")** : 데이터베이스에 생성될 테이블 이름입니다.
>
> **@GeneratedValue(strategy = GenerationType.IDENTITY)** 테이블의 기본키 생성기능을 우리가 연결한 DB에서 처리할 수 있도록 설정하는 부분입니다. (우리는 지금 예제에서는 사용하지 않도록 합니다.)
>
> **toDomain()** : BookJpo객체를 Book객체로 변환하는 메서드 입니다.
>
> **toDomains(List<BookJpo> bookJpos)** : BookJpo리스트를 Book리스트로 변환하는 메서드입니다.

JPA를 이용하여 mariaDB에서 Books테이블에 대한 정보를 BookJpo객체로 받아온다음 실제 값을 넘겨줄때에는 Book객체 형태로 넘겨주기 위함입니다.

### 5-6. BookStoreJPARepository 생성

이번에는 간단한 CRUD를 구현하기 때문에 JpaRepository를 상속받아 미리 구현된 메서드를 이용해 보려고합니다.

#### books/store/repository/BookStoreJPARepository.java

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookStoreJPARepository extends JpaRepository<BookJpo, String>{

}
```

> **JpaRepository**에는 우리가 위에서 만든 **BookJpo**객체 타입과 Id값의 타입인 **String**을 넣어줍니다.

### 5.7 BookStore 구현

위에서 정의한 BookStore Interface를 구현합니다.

#### books/store/logic/BookStoreJPAImpl.java

```java
@RequiredArgsConstructor
@Repository
public class BookStoreImpl implements BookStore{

    //BookStoreJPARepository 주입
	private final BookStoreJPARepository bookRepository;

	@Override
	public List<Book> retrieveAll() throws NoSuchElementException {
		List<BookJpo> bookJpos = bookRepository.findAll();
		if(bookJpos.isEmpty())throw new NoSuchElementException("book empty");
		//데이터가 존재하는 경우에만 Return 이때, BookJpo리스트를 book리스트로 변경해줍니다.
		return BookJpo.toDomains(bookJpos);
	}

	@Override
	public Book retrieve(String ISBN) throws NoSuchElementException {
		BookJpo bookJpo = bookRepository.findById(ISBN).orElseThrow(() -> new NoSuchElementException());
		//데이터가 존재하는 경우에만 Return 이때, BookJpo객체를 book객체로 변경해줍니다.
		return bookJpo.toDomain();
	}

	@Override
	public void create(Book book) throws DuplicateException {
		if(isExist(book.getISBN())) throw new DuplicateException(book.getISBN());
		//테이블에 해당 book이 존재하지 않는 경우에만 Update
        //실제 DB에 객체로 요청을 보낼때에는 BookJpo객체의 형태로 보내주어야 합니다.
		bookRepository.save(new BookJpo(book));
	}

	@Override
	public void update(Book book) throws NoSuchElementException {
		if(!isExist(book.getISBN())) throw new NoSuchElementException(book.getISBN());
		//테이블에 해당 book이 존재하는 경우에만 Update
        //실제 DB에 객체로 요청을 보낼때에는 BookJpo객체의 형태로 보내주어야 합니다.
		bookRepository.save(new BookJpo(book));
	}

	@Override
	public void delete(String ISBN) throws NoSuchElementException {
		if(!isExist(ISBN)) throw new NoSuchElementException(ISBN);
		//테이블에 해당 book이 존재하는 경우에만 Delete
		bookRepository.deleteById(ISBN);
	}

	@Override
	public boolean isExist(String ISBN) {
		Optional<BookJpo> book = bookRepository.findById(ISBN);
		//Optional안에 bookJpo객체가 존재하는 경우
		if(bookOpt.isPresent()) return true;

		//Optional안에 bookJpo객체가 존재하지 않는 경우
		return false;
	}
}
```

> **@RequiredArgsConstructor** : final 필드나, @NonNull 이 붙은 필드에 대해 생성자를 생성해 준다.
>
> **BookStoreJPARepository** : 생성자를 통한 의존성 주입을 이용하여 의존성을 주입해줍니다.
>
> **isExist(String ISBN)** : 실제 테이블에 값이 존재하는지 확인해줍니다. (있으면 : true / 없으면 : false)
>
> **isPresent()** : Optional안에 값이 존재하는지 확인합니다. (존재하면 true : 존재하지 않으면 false)

우리만의 Exception을 만들기위해 DuplicateException을 생성합니다.

#### books/store/exceptions/DuplicateException.java

```java
public class DuplicateException extends Exception {

	public DuplicateException() {
		super("already exist");
	}

	public DuplicateException(String massage) {
		super(massage+" already exist ");
	}
}
```

> **DuplicateException()** :최상위 Exception인 Exception클래스를 상속받고 부모의 생성자를 Override합니다.
>
> **DuplicateException(String massage)** : 생성자를 오버로딩 해서 파라미터값을 넣어주었을 때의 메세지가 다르게 보여지도록 수정합니다.

### 5-8. Service 구현

위에서 생성한 Service Interface구현체인 ServiceIml을 구현합니다.

#### books/domain/service/logic/BookServiceImpl.java

```java
@RequiredArgsConstructor
@Service
public class BookServiceImpl implements BookService{
	// 만들어야하는 메서드를 실제 구현 하는 클래스

    //BookStore 주입
	private final BookStore bookStore;

	@Override
	public List<Book> bookList() throws NoSuchElementException{
		return bookStore.retrieveAll();
	}

	@Override
	public Book bookDetail(String ISBN) throws NoSuchElementException {
		return bookStore.retrieve(ISBN);
	}

	@Override
	public void bookCreate(Book book) throws DuplicateException {
		bookStore.create(book);
	}

	@Override
	public void bookModify(Book book) throws NoSuchElementException {
		bookStore.update(book);
	}

	@Override
	public void bookDelete(String ISBN) throws NoSuchElementException {
		bookStore.delete(ISBN);
	}
}
```

> **BookStore** : 우리가 만든 JPA를 이용하여 요청하는 BookStore객체를 주입해줍니다.
>
> 각 서비스의 메소드에서 필요한 JPA요청을 수행해 줍니다.

### 5-9. REST Controller 구현

마지막으로 Controller를 구현하여 서버구동시 요청한 Url에 맞는 기능을 수행할 수 있도록 한다.

#### books/application/BookRESTController.java

```java
@RequiredArgsConstructor
@RestController
@RequestMapping("/api")
public class BookRESTController {

	private final BookService service;

	@GetMapping("/books/")
	public List<Book> getBooks(){
		return service.bookList();
	}

	@GetMapping("/books/{ISBN}/")
	public Book getBook(@PathVariable String ISBN){
		return service.bookDetail(ISBN);
	}

	@PostMapping("/books/")
	public void insertBook(@RequestBody Book book) throws DuplicateException {
		service.bookCreate(book);
	}

	@PutMapping("/books/")
	public void updateBook(@RequestBody Book book){
		service.bookModify(book);
	}

	@DeleteMapping("/books/{ISBN}/")
	public void deleteBook(@PathVariable String ISBN){
		service.bookDelete(ISBN);
	}

    @ExceptionHandler(RuntimeException.class)
	public @ResponseBody ErrorMessage runTimeError(RuntimeException e) {
		ErrorMessage error = new ErrorMessage();
		error.setMessage(e.getMessage());
		return error;
	}
}
```

> **@RestController** : 간단히는 ( @Controller + @ResponseBody ) return시 **Json 형태**로 객체 데이터를 반환
>
> **@RequestMapping("/api")** : 해당 컨드롤러에 기본적인 URL매핑을 적용합니다 ( /api/ 를 기본으로)
>
> **@GetMapping** : GET 요청을 했을 때 요청한 Url에 맞는 메서드를 수행합니다.
>
> **@PostMapping** : POST요청을 했을 때 요청한 Url에 맞는 메서드를 수행합니다.
>
> **@PutMapping** : PUT 요청을 했을 때 요청한 Url에 맞는 메서드를 수행합니다.
>
> **@DeleteMapping** : DELETE 요청을 했을 때 요청한 Url에 맞는 메서드를 수행합니다.
>
> **@ExceptionHandler** : @Controller, @RestController가 적용된 Bean내에서 발생하는 예외를 잡아서 하나의 메서드에서 처리해주는 기능을 합니다.

위에서 사용된 ErrorMessage 객체를 만들어 봅시다.

#### books/application/ErrorMessage.java

```java
@Data
public class ErrorMessage {
	private String message;
}
```

> **@Data** : @Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode을 한번에 해주는 어노테이션 입니다.
>
> **\* lombok 참고**
>
> - https://projectlombok.org/features/GetterSetter
> - https://projectlombok.org/features/constructor
> - https://projectlombok.org/features/ToString
> - https://projectlombok.org/features/EqualsAndHashCode
> - https://projectlombok.org/features/Data

### 5-10. 테스트

Postman을 이용하여 우리가 만든 book Api가 정상적으로 동작하는지 확인 해 보겠습니다.

#### GET /api/books/

![LIST](/images/project/react_spring/part3/LIST.png)

#### GET /api/books/{ISBN}/

![DETAIL](/images/project/react_spring/part3/DETAIL.png)

#### POST /api/books/

![POST](/images/project/react_spring/part3/POST.png)

연결된 로컬 DB에서 데이터가 생성된 것 확인

![POST_결과](/images/project/react_spring/part3/POST_결과.png)

#### PUT /api/books/

![UPDATE](/images/project/react_spring/part3/UPDATE.png)

#### DELETE /api/books/{ISBN}/

![DELETE](/images/project/react_spring/part3/DELETE.png)

다음에는 React로 만든 Book front페이지와 이번에 만든 Book Api를 서로 연결시켜 프론트부분이 데이터를 서버에서 주고받을 수 있도록 구현해보겠습니다.
