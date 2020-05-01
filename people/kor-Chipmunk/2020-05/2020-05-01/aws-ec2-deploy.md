# EC2 서버에 프로젝트를 배포해 보자

## 스프링 부트 프로젝트로 RDS 접근하기

스프링 부트 프로젝트에서 MariaDB를 사용하기 위해선 몇 가지 작업이 필요하다.

- 테이블 생성
  - H2에서 자동 생성해주던 테이블들을 MariaDB에선 직접 쿼리를 이용해 생성한다.
- 프로젝트 설정
  - 자바 프로젝트가 MariaDB에 접근하려면 데이터베이스 드라이버가 필요하다. MariaDB에서 사용 가능한 드라이버를 프로젝트에 추가한다.
- EC2 (리눅스 서버) 설정
  - 외부에서 보여지지 않도록 EC2 서버 내부에서 접속 정보를 관리하도록 설정한다.

### RDS 테이블 생성

우선 RDS에 테이블을 생성한다. JPA가 사용될 엔티티 테이블과 스프링 세션이 사용될 테이블 2가지 종류를 생성한다. JPA가 사용할 테이블은 테스트 코드 수행 시 로그로 생성되는 쿼리를 사용한다.

테스트 코드를 수행하면 다음과 같은 오류 로그가 발생한다. create table 부터 구문을 복사해 RDS에 반영한다.

```sql
Hibernate: create table posts (id bigint not null auto_increment, created_date datetime, modified_date datetime, author varchar(255), content TEXT not null, title varchar(500) not null, primary key (id)) engine=InnoDB

Hibernate: create table user (id bigint not null auto_increment, created_date datetime, modified_date datetime, email varchar(255) not null, name varchar(255) not null, picture varchar(255), role varchar(255) not null, primary key (id)) engine=InnoDB
```

스프링 세션 테이블은 schema-mysql.sql 파일에서 확인할 수 있다. 인텔리제이의 File 검색 기능(Ctrl+Shift+N, Command+Shift+O)으로 찾는다. 해당 파일에 세션 테이블 SQL 구문들이 있다. 이 구문들을 복사해 RDS에 반영한다.

위 작업으로 RDS에 필요한 테이블은 모두 생성헀다.

### 프로젝트 설정

MariaDB 드라이버를 build.gradle에 등록한다.

```
compile("org.mariadb.jdbc:mariadb-java-client")
```

서버에서 구동될 환경을 하나 구성한다.

`src/main/resources`에 `application-real.properties` 파일을 추가한다. `profile=real`인 환경이 구성된다. 보안/로그상 이슈가 될 만한 설정들을 모두 제거하여 RDS 환경 profile 설정으로 만든다.

```properties
spring.profiles.include=oauth,real-db
spring.jpa.properties.hinernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```

변경 이력들을 모두 커밋해 깃허브로 푸쉬한다.

### EC2 설정

OAuth와 마찬가지로 RDS 접속 정보도 보호해야 한다. EC2 서버에 직접 설정 파일을 만든다.

app 디렉토리에 `application-real-db.properties` 파일을 생성한다.

```sh
vim ~/app/application-real-db.properties
```

다음과 같은 내용을 추가한다.

```properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://rds주소:포트명(기본은 3306)/database이름
spring.datasource.username=db계정
spring.datasource.password=db계정 비밀번호
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```

1. `spring.jpa.hibernate.ddl-auto=none`
   1. JPA로 테이블이 자동 생성되는 옵션을 생성하지 않음으로 지정한다.
   2. RDS에는 실제 운영으로 사용될 테이블이다. 절대 스프링 부트에서 새로 만들지 않도록 해야한다.
   3. 이 옵션을 하지 않으면 자칫 테이블이 모두 새로 생성될 수 있다.
   4. 주의해야 하는 옵션이다.

마지막으로 deploy.sh가 real profile을 쓸 수 있도록 다음과 같이 변경한다.

```sh
...
nohup java -jar \
   -Dspring.config.location=classpath:/application.properties, /home/ec2-user/app/application-oauth.properties,
   /home/ec2-user/app/application-real-db.properties \
   -Dspring.profiles.active=real \
   $REPOSITORY/$JAR_NAME 2>&1 &
```

1. `-Dspring.profiles.active=real`
   1. application-real.properties를 활성화시킨다.
   2. application-real.properties의 spring.profiles.inclue=oauth,real-db 옵션 때문에 real-db 역시 함께 활성화 대상에 포함된다.

nohup.out 파일에 다음 로그가 보인다면 성공적으로 수행된 것이다.

```sh
Tomcat started on port(s): 8080 (http) with context path ''
Started Application in ~~ seconds (JVM running for ~~~)
```

cmd 명령어로 html 코드가 정상적으로 보인다면 성공이다.

```sh
curl localhost:8080
```

# 참고 도서

스프링 부트와 AWS로 혼자 구현하는 웹 서비스