# Spring boot Docker로 Maria DB 사용하기

기존방식처럼 로컬에 DB를 깔아서 사용하는 방법 말고, 도커 이미지를 이용하여 컨테이너로 DB를 실행하고, 실행되어있는 DB와 스프링부트를 연결을 해보자!



### 1. mariadb 이미지 받기

```shell
docker pull mariadb:latest
```



### 2. 이미지파일을 컨테이너로 실행

받은 mariadb 이미지 파일을 실행할 때 mariadb의 유저와 DATABASE환경설정을 같이 해준다.

```shell
docker run -p 30000:3306 --name maria_boot -e MYSQL_ROOT_PASSWORD=1234 -e MYSQL_DATABASE=spring_db -e MYSQL_USER=spring -e MYSQL_PASSWORD=spring -d mariadb
```

>**-p 30000:3306** : 위 명령대로 실행하면, 외부에서 Docker host 의 30000포트로 요청이 들어오면 maria_boot컨테이너의 3306 포트로 해당 요청을 forwarding 하겠다는 의미(30000포트는 다른포트와 겹치지 않으려고 임의로 설정)
>
>**--name maria_boot** : 컨테이너 이름을 maria_boot로 설정
>
>**MYSQL_ROOT_PASSWORD** : mariadb의 password 를 1234로 설정
>
>**MYSQL_DATABASE** : mariadb의 db 이름을 spring_db로 설정
>
>**MYSQL_USER** : mariadb의 User이름을 spring으로 설정
>
>**MYSQL_PASSWORD** : mariadb의 password를 spring으로 설정



### 3. mariadb를 사용하기위한 dependency 설치

##### pom.xml

```xml
<!-- mariadb -->
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
</dependency>
```



### 4. 스프링부트 yml파일 수정

스프링부트에서 우리가 실행한 mariadb컨테이너와 연결할 수있도록 yml파일을 설정한다.

##### application.yml

```yaml
spring: 
# Maria DB
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver 
    url: jdbc:mariadb://localhost:30000/spring_db
    username: spring
    password: spring
  jpa:
    database-platform: org.hibernate.dialect.MariaDB53Dialect
    hibernate:
      ddl-auto: create
```

