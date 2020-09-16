# Dialect이란?

SQL은 표준 ANCI SQL이 있고 DBMS Vendor(공급업체)인 MS-SQL, Oracle, My-SQL, Postgre SQL에서 제공하는 SQL이 존재합니다.

MS-SQL은 T-SQL, Oracle은 PL/SQL이 대표적입니다.
ANSI SQL은 모든 DBMS에서 공통적으로 사용이 가능한 표준 SQL이지만 DBMS에서 만든 SQL은 자신들만의 독자적인 기능을 추가 하기 위해 만든 것으로 사용하는 DBMS에서만 사용이 가능합니다.

JPA는 기본적으로 어플리케이션에서 직접 JDBC 레벨의 SQL을 작성하지 않고 JPA가 직접 SQL을 작성하고 실행합니다. 그런데 DBMS 종류마다 사용하는 SQL이 다르다는 것을 우리는 앞에서 알아봤습니다. JPA가 해당 DBMS에 맞춰 SQL을 생성해야하는데 어떤 종류인지 알지 못한다면 문제가 발생 할 수 있습니다.
그래서 JPA에 어떤 DBMS를 사용하는 알려주는 방법이 방언을 설정하는 방법입니다. JPA에 Dialect를 설정할 수 있는 추상화 방언 클래스를 제공하고 설정된 방언으로 각 DBMS에 맞는 구현체를 제공합니다.

![Dialect](/images/spring/Dialect.png)

그래서 만약, MariaDB로 개발을 진행하면 Dialect에 MariaDB로 설정해 놓다가 어플리케이션을 설치할때 DBMS가 Oracle 이라면 Dialect을 Oralce로 설정하면 문제 없이 어플리케이션을 구동 할 수 있습니다.

MariaDB 10.3 버전 사용시

```
database-platform: org.hibernate.dialect.MariaDB103Dialect
```

Oracle 11g 버전 사용시

```
database-platform: org.hibernate.dialect.Oracle10gDialect
```

JPA에서는 설정을 통해 원하는 Dialect만 설정해주면 해당 Dialect를 참고하여 그에 맞는 쿼리를 생성합니다. 따라서 개발시에는 Oracle DB에 맞게 설정하고 어플리케이션을 개발하다가 실제 고객의 환경이 SQL SERVER를 사용중이라면 설정만 SQLServerDialect로 변경해 줌으로써 불필요한 변경에 대한 자원 소모를 줄일 수 있습니다.
