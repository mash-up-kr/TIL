# @RequestBody, @ModelAttribute, @RequestParam

## 1. @RequestBody

클라이언트가 전송하는 HTTP 요청의 Body 내용을 Java Object로 변환시키는 역할을 한다.  
`@RequestBody`는 HTTP POST 메소드 요청과 함께 사용되어야 한다.  
Body가 존재하지 않는 HTTP GET 메소드에 `@RequestBody`를 쓰는 것은 적합하지 않아 오류가 발생한다.  
`@RequestBody`는 JSON이나 XML과 같은 형태의 데이터를 `Jackson` 등의 `MessageConverter`로 변환해 Java Object로 변환한다.  
데이터와 자바 객체를 1:1로 매핑되어서 변환되는 `@ModelAttribute`와 차이점이 있다.  
즉, `@RequestBody`는 POST 방식으로 JSON의 형태로 넘겨온 데이터를 객체로 바인딩하기 위해 사용한다.

## 2. @ModelAttribute

`@ModelAttribute`는 클라이언트가 전송하는 여러 파라미터들을 일대일로 객체에 바인딩해 다시 뷰로 전달해주는 객체다.  
`@ModelAttribute`는 매핑시키는 파라미터의 타입이 객체의 타입과 일치하는지를 검증을 해준다. 이 외에도 다양한 검증 작업이 추가적으로 진행된다. int형 변수에 String형을 넣는다면, `BindException` 예외가 발생한다. `@RequestBody`는 JSON이나 XML을 Jackson과 같은 `MessageConverter`를 사용해 변환시키지만, `ModelAttribute`는 여러 개의 파라미터를 바로 자바빈 객체로 매핑시킨다.

따라서 `@ModelAttribute`는 JSP에서 `Form` 태그로 전달받은 파라미터들을 객체로 바인딩 시키는 경우에 사용한다.

참고로 `@ModelAttribute`는 바인딩 시키는 객체에 Setter 함수가 없다면 매핑이 되지 않는다. `@RequestBody`는 요청 받은 데이터를 변환하는 것이기에, Setter 함수가 없어도 값이 매핑이 된다.

`@ModelAttribute` 어노테이션 속성으로 특정 파라미터만을 받을 수 있다.

```json
{
    writer: '작가',
    contents: '내용'
}
```
의 형태의 데이터를 전송했다고 가정하자.  
컨트롤러는 `@ModelAttribute('writer') String writer`의 형태로 `writer` 변수에 `'내용'`만을 바인딩 시킬 수 있다.

## 3. @RequestParam

`@RequestParam`은 요청 파라미터를 메소드에서 일대일로 받기 위해 사용된다. `@RequestParam`를 사용하면 기본적으로 반드시 해당 파라미터가 전송 되어야만 한다. 만약 그렇지 않으면 `400 Error` 응답 코드를 유발한다. 반드시 필요한 변수가 아니라면 `required` 속성을 `false` 값으로 설정해둔다. 해당 파라미터를 사용하지 않고 요청을 보낼 경우 `default`로 받을 값을 설정할 수도 있다. 즉, `defaultValue`로 기본값을 설정할 수 있고 필수 요구사항이라면, `required` 속성으로 설정할 수 있다.

## 활용 예시

Model 객체 `BoardVO`는 다음과 같다.

```java
public class BoardVO {

    private int index;
    private String writer;
    private String contents;

    ...
    (getter, setter)
}
```

컨트롤러에서 다음과 같이 사용할 수 있다.

```java
@Controller
@RequestMapping(value = "/board")
@Log4j2
public class BoardController {

    @Resource(name = "boardService")
    private BoardService boardService;

    @PostMapping(value = "/requestBody")
    public ResponseEntity requestBody(@RequestBody BoardVO boardVO){
        // @RequestBody는 Json으로 받은 요청을 MessageConverter를 통해 Java 객체로 변환시킨다.
        log.info(boardVO.getContents());
        return ResponseEntity.ok(boardService.add(boardVO));
    }

    @PostMapping(value = "/modelAttribute")
    public String modelAttribute(Model model, @ModelAttribute BoardVO boardVO){
        // @ModelAttribute는 Form태그를 통해 전송받은 파라미터들을 Java 객체로 매핑시킨다.
        // 만약 boardVO에 contents에 대한 Setter함수가 없다면 매핑을 시키지 못하고, 항상 null을 갖게 된다.
        log.info(boardVO.getContents());
        model.addAttribute("boardVO", boardService.add(boardVO));
        return "/views/board/detail";
    }

    @GetMapping(value = "/list")
    public ResponseEntity requestParam(@RequestParam(value = "searchKeyWord1", required = false) String searchKeyWord1, @RequestParam(value = "writer", defaultValue = "Writer") String searchKeyWord2){
        List<BoardVO> boardList;

        // searchKeyWord1는 required가 false이기 때문에 없을 수도 있다.
        if(searchKeyWord1 != null){
            boardList = boardService.findAllBoardBySearchKeyWord(searchKeyWord1);
        } else {
            boardList = boardService.findAllBoard();
        }
        // 반면에, searchKeyWord2는 required가 true이기 때문에 반드시 요청 파라미터로 존재해야 한다.
        log.info(searchKeyWord2);

        return ResponseEntity.ok(boardList);
    }

}
```

`@RequestBody`는 JSON으로 받은 요청을 `MessageConverter`가 자바 객체로 변환을 시켜주기에 Setter 함수가 없어도 정상적으로 변수들이 저장된다.

`@ModelAttribute`는 전달받은 파라미터들을 그대로 자바 객체로 매핑시켜준다. 그 때문에 변수들의 Setter 함수가 없다면 정상적으로 저장되지 않는다.

`@RequestParam`의 경우 하나의 파라미터를 얻기 위해 사용한다. 해당 파라미터가 꼭 필요한 것이 아니라면 `required=false` 속성으로 넣는다.

## 참고 링크

1. [https://mangkyu.tistory.com/72]()