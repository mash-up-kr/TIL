# 스프링 부트 교재 복습

## 1. @SpringBootApplication

`@SpringBootApplication` 어노테이션은 세 가지 특징을 제공한다.

1. `@EnableAutoConfiguration` : Spring Boot의 자동 설정 기능을 활성화한다.
2. `@ComponentScan` : 패키지 내 `Application` 컴포넌트가 어디에 위치했는지 검사한다. ( Bean 검색 )
3. `@Configuration` :  Context에 Bean을 추가하거나 어떤 클래스를 참조할 수 있다.

## 2. @RestController

컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.  
`@ResponseBody`를 각 메소드마다 선언했던 것을 한 번에 사용할 수 있게 해준다.

Spring에서 컨트롤러를 지정해주기 위한 어노테이션은 `@Controller`와 `@RestController`가 있다.  
전통적인 Spring MVC의 컨트롤러인 `@Controller`와 Restful 웹서비스의 컨트롤러인 `@RestController`의 주요 차이점은 HTTP Reponse Body가 생성되는 방식이다.

`@Controller`는 주로 뷰를 반환하기 위해 사용된다. 클라이언트의 요청을 다음의 과정으로 처리한다.
1. 클라이언트는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. 매핑되는 핸들러와 그 타입을 찾는 `DispatcherServlet`이 요청을 인터셉트한다.
3. Controller가 요청을 처리한 후에 응답을 DispacherServlet 으로 반환되고, DispatcherServlet은 뷰를 사용자에게 반환한다.

컨트롤러에서 뷰가 아닌 데이터를 반환해줘야 하는 경우가 있다.  
데이터를 반환하기 위해 `@ResponseBody` 어노테이션을 활용해야 한다.  
이를 통해 Controller에서 JSON 형태로 데이터를 반환할 수 있다.

1. 클라이언트는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. 매핑되는 핸들러와 그 타입을 찾는 DispatcherServlet이 요청을 인터셉트한다.
3. @ResponseBody를 사용하여 클라이언트에게 Json 형태로 데이터를 반환한다.

`@Controller`와 `@ResponseBody`를 사용한 예제는 다음과 같다.

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @Resource(name = "userService")
    private UserService userService;

    @RequestMapping(value = "/retrieveUserInfo", method = RequestMethod.POST)
    public @ResponseBody UserVO retrieveUserInfo(@RequestBody UserVO userVO){
        return userService.retrieveUserInfo(userVO);
    }
    
    @RequestMapping(value = "/userInfoView", method = RequestMethod.POST)
    public String loginUserView(@RequestBody UserVO userVO, Model model){
        model.addAttribute("userInfo", userVO);
        return "/user/userInfoView";
    }

}
```

`@RestController`와 Spring MVC Controller에 `@ResponseBody`가 추가되었다.  
주 용도는 JSON/XML 형태로 객체 데이터를 반환할 때다. 주로 API 서버로 활용할 때 쓰인다.

1. 클라이언트는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. 매핑되는 핸들러와 그 타입을 찾는 DispatcherServlet이 요청을 인터셉트한다.
3. RestController는 해당 요청을 처리하고 데이터를 반환한다.

`@RestController`의 예는 다음과 같다.

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Resource(name = "userService")
    private UserService userService;

    @RequestMapping(value = "/retrieveUserInfo", method = RequestMethod.POST)
    public UserVO retrieveUserInfo1(@RequestBody UserVO userVO){
        return userService.retrieveUserInfo(userVO);
    }

    @RequestMapping(value = "/retrieveUserInfo", method = RequestMethod.POST)
    public ResponseEntity<UserVO> retrieveUserInfo2(@RequestBody UserVO userVO){
        userVO = userService.retrieveUserInfo(userVO);

        if(userVO == null){
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }

        return new ResponseEntity<>(userVO, HttpStatus.OK);
    }

}
```

## 참고 링크

1. https://mangkyu.tistory.com/49