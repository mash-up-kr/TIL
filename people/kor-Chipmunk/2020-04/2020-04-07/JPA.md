JPA는 크게 객체와 테이블을 어떻게 매핑해야 하는지에 관한 설계와 설계한 모델을 실제 사용하는 부분으로 나눌 수 있다.

## ORM ( Object-Relational Mapping ) 프레임워크

객체와 관계형 데이터베이스 간의 차이를 중간에서 해결해주는 프레임워크다. JPA는 자바 진영의 ORM 기술 표준이다.

## SQL을 직접 다룰 때 발생하는 문제점

데이터베이스에 데이터를 삽입하고 관리하려면 SQL 을 사용한다.  
자바로 작성한 애플리케이션은 JDBC API로 SQL을 데이터베이스에 전달한다.

### 1. 코드 반복

JDBC API로 데이터베이스의 데이터를 관리할 때 많은 코드가 중복되며 반복된다.

### 2. SQL에 의존적

데이터베이스의 테이블이 많아지고 애플리케이션의 크기가 많아질수록 SQL 구문의 수정을 위한 코드 수정이 번거롭고 실수할 위험이 크다.

SQL 구문을 실수한다면 객체에 매핑이 제대로 안될 수도 있다.

이러한 문제를 고칠 시 DAO로 추상화 시켜도 결국 SQL 구문을 쓴 파일을 열어보고 확인할 수 밖에 없다. 즉, 개발자는 엔티티를 신뢰할 수 없게 된다.

엔티티(Entity) : 비즈니스 요구사항을 모델링한 객체

## JPA는 어떻게 이 문제를 해결했는가

JPA로 데이터를 데이텉베이스로 저장할 때 SQL를 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다.

JPA가 제공하는 CRUD API는 다음과 같다.

### 1. 저장 기능

```java
jpa.persist(user); // 저장
```

JPA에서 객체와 매핑 정보를 보고 적절한 INSERT SQL을 생성한다. 매핑 정보는 어떤 객체를 어떤 테이블로 매핑할 지를 정의한 정보다.

### 2. 조회 기능

```java
String userId = "kor-Chipmunk";
User user = jpa.find(User.class, userId); // 조회
```

적절한 SELECT SQL을 생성하고 그 결과를 `User` 객체를 생성해 반환한다.

### 3. 수정 기능

```java
User user = jpa.find(User.class, userId);
user.setName("다람쥐");
```

JPA는 따로 수정 메소드를 제공하지 않는다. 그 대신 객체의 값을 변경하면 트랜잭션을 커밋할 때 적절한 UPDATE SQL을 전달한다.

### 4. 연관 객체 조회 기능

```java
User user = jpa.find(User.class,, userId);
Group group = user.getGroup(); // User 객체와 연관된 객체 조회
```

연관된 객체도 SELECT SQL 구문을 생성해 조회가 가능하다.

## 참고 도서

자바 ORM 표준 JPA 프로그래밍