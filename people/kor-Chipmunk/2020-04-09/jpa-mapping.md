## JPA 객체 매핑

테이블 생성
```sql
CREATE TABLE MEMBER (
ID VARCHAR(255) NOT NULL,
NAME VARCHAR(255),
AGE INTEGER NOT NULL,
PRIMARY KEY (ID)
)
```

Member 객체를 생성하여 테이블과 매핑한다.

```java
package jpabook.start;

import javax.persistence.*;  //**

/**
 * User: HolyEyE
 * Date: 13. 5. 24. Time: 오후 7:43
 */
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

매핑 정보를 표시할 때 어노테이션을 사용한다.  
JPA 어노테이션의 패키지는 `javax.persistence` 에 있다.

**1. @Entity**  
해당 클래스를 테이블과 매칭한다는 정보다. 이 어노테이션이 적용된 클래스를 엔티티 클래스라고 부른다.

**2. @Table**  
엔티티 클래스에 매핑할 테이블 정보다. name 속성을 생략한다면 클래스 이름이 테이블 이름과 매핑된다.

**3. @Id**  
엔티티 클래스의 필드를 테이블의 기본 키로 매핑한다. 이 어노테이션이 적용된 필드를 식별자 필드라 한다.

**4. @Column**  
필드를 테이블 칼럼에 매핑한다. name 속성으로 해당하는 테이블 칼럼과 매핑되게 한다.

**매핑 정보가 없는 필드**  
생략 시 필드명으로 칼럼명과 매핑된다. DB에서 대소문자를 구분하지 않는다고 가정한다. 대소문자를 구분하면 정확한 칼럼 이름과 매핑해야 한다.

## 참고 자료 및 참고 도서

1. 자바 ORM 표준 JPA 프로그래밍, 김영한
2. 예제 소스 : https://github.com/holyeye/jpabook