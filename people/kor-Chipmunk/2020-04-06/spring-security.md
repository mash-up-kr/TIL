# Spring Security를 Spring Boot에 적용하기

## Spring Security

스프링 시큐리티는 인증 기능과 인가 기능을 가진 프레임워크다.  
스프링 기반 애플리케이션에서는 보안 표준이다.  
인터셉터, 필터 기반의 보안 기능을 구현할 수도 있지만 스프링 시큐리티로 구현하는 것을 권장한다.  
  
스프링 시큐리티와 OAuth 2.0을 구현한 소셜 로그인 기능을 만들 수 있다.

[Spring Security 공식 문서 바로 가기](https://spring.io/projects/spring-security#overview)

## 구글 서비스 등록

[구글 클라우드 플랫폼](https://console.cloud.google.com) 에서 서비스를 등록할 수 있다.  
Spring boot 프로젝트의 `application-oauth.properties` 에 `client-id`와 `client-secret`, `scope` 를 등록한다.

```
spring.security.oauth2.client.registration.google.client-id={client-id}
spring.security.oauth2.client.registration.google.client-secret={client-secret}
spring.security.oauth2.client.registration.google.scope=profile,email
```

`application-xxx.properties` 을 설정 파일로 불러오려면,
`application.properties` 파일에서 다음 코드를 추가한다.
```
spring.profiles.include=oauth
```

키 보안을 위해 `.gitignore` 파일에서 `application-oauth.properties` 파일을 제외해야 한다.

## 구글 로그인 연동하기

`domain.user.User` 클래스를 만든다.  
사용자 정보를 담당할 도메인이다.
```java
package com.chipmunk.boot.springboot.domain.user;

import com.chipmunk.boot.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }

}
```

`BaseTimeEntity` 는 `createdDate`와 `modifiedDate` 속성을 추가해준다. `domain.BaseTimeEntity` 클래스를 만든다.

```java
package com.chipmunk.boot.springboot.domain;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modifiedDate;

}
```

`@Enumerated(EnumType.STRING)` 어노테이션은 JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정한다.  
기본으로 정수형 int 의 숫자가 저장된다.  
숫자보다 Enum 값을 문자열로 저장할 수 있도록 선언한다.

`domain.user.Role` Enum 클래스는 다음과 같다.

```java
package com.chipmunk.boot.springboot.domain.user;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;

}
```

스프링 시큐리티에서의 권한 코드는 항상 ROLE_ 접두사가 있어야 한다.  

User의 CRUD를 담당하는 `UserRepository` 클래스는 다음과 같다.
```java
package com.chipmunk.boot.springboot.domain.user;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

}
```

## 스프링 시큐리티 설정

스프링 시큐리티 의존성 설정은 다음과 같다.
`compile('org.springframework.boot:spring-boot-starter-oauth2-client')`

`spring-security-oauth2-client`와 `spring-security-oauth2-jose` 를 기본으로 관리한다.  
스프링 시큐리티에서 소셜 로그인을 구현할 때 필요하다.

`config.auth.SecurityConfig` 클래스를 생성한다.

```java
package com.chipmunk.boot.springboot.config.auth;

import com.chipmunk.boot.springboot.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```

`@EnableWebSecurity` 어노테이션은 Spring Security 설정들을 활성화시켜준다.  

`csrf().disable().headers().frameOptions().disable()` 설정은 h2-console 화면을 사용하기 위해 보안 옵션을 해제한다.  

`authorizeRequests()` 는 URL별 권한 관리를 설정하는 함수를 사용할 수 있다. `antMatchers()` 함수를 사용할 수 있다.

`antMatchers()` 함수는 URL, HTTP 메소드 별로 권한을 관리할 수 있다. `permitAll()` 함수는 모두 열람이 가능한 권한을 준다. `hasRole()` 함수로 특정 권한을 가져야만 열람할 수 있다.

`anyRequest()` 함수는 그 외 URL 들을 포함한다.  
주소 요청을 정의하고 그 뒤에 어떻게 접근하는지를 정의할 수 있다.

* `anonymous()` : 인증되지 않은 사용자가 접근할 수 있다.
* `authenticated()` : 인증된 사용자만 접근할 수 있다.
* `fullyAuthenticated` : 완전하게 인증된 사용자만 접근할 수 있다.
* `hasRole()`, `hasAnyRole()` : 특정 권한이 있는 사용자만 접근할 수 있다.
* `hasAuthority()`, `hasAnyAuthority()` : 
* `hasIpAddress()` : 특정 아이피 주소를 가지는 사용자만 접근할 수 있다.
* `permitAll()`, `denyAll()` : 접근을 전부 허용하거나 거부한다.
* `rememberMe()` : 리멤버 기능으로 로그인한 사용자만 접근할 수 있다.

### Authenticated() vs FullyAuthenticated()

|Authenticated Function|Description|
|---|---|
|`isAuthenticated()`|Returns true if the user is **not anonymous**|
|`isFullyAuthenticated()`|Returns true if the user is **not an anonymous** or a **remember-me user**|

