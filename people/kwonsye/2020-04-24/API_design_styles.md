## API 디자인 스타일

### REST
- 가장 보편화된 web API design
- Roy Fielding 에 의해 처음 제시됨
- **stateless한 상태에서 텍스트 기반 리소스**로 상호작용하는 방식
- GET, POST, PUT, DELETE operation
- **Hypermedia rich**
    - hypermedia를 지원하지 않으면 RESTful한API가 아니다.
- 클라이언트와 서버가 약하게 결합됨
- layered architecture, efficient caching, and high scalability에서 REST는 좋은 선택!


```
curl -v -X GET https://api.sandbox.paypal.com/v1/activities/activities?start_time=2012-01-01T00:00:01.000Z&amp;end_time=2014-10-01T23:59:59.999Z&amp;page_size=10 \
-H "Content-Type: application/json" \
-H "Authorization: Bearer Access-Token"
```

### GraphQL
- 전통적인 방식과는 다름
- 클라이언트가 스스로 **어떤 데이터를 원하고, 어떤 방식과 형식으로 원하는지** 설명함  
- 가능한 가장 작은 request를 보낼 수 있음
- 필요한 데이터 타입이 잘 정의되어 있고, low data package 가 선호될 때 사용하면 좋다.
- REST 보다 더 나은 방법이라고 생각하지 말고, 클라이언트와 데이터의 관계를 새롭게 바라보는 다른 하나의 옵션이라고 생각해라.
- Github 에서 사용
    - REST 는 클라이언트가 원하지 않는  데이터까지 주므로 유연하지 못하다. 


### Webhooks
- 매우 심플하게 이벤트가 발생하면 HTTP POST가 실행된다.
- 클라이언트가 request를 보내고 서버가 response하는 전통적인 서버-클라이언트와 반대됨
- 서버는 프로비저닝 된 리소스를 **업데이트 한 다음 업데이트 된 데이터를 클라이언트에 자동으로 전송**한다.
- 클라이언트는 요청하는 자가 아니고, **수동적인 리시버**이다.
- 클라이언트가 직접 요청하지 않고 데이터를 받기만 하므로
    - 원격 코드베이스를 **업데이트하고 리소스를 쉽게 배포**할 수 있으며 
    - 기존 시스템에 통합하여 API 관련 엔드 포인트 및 기타 데이터를 업데이트 할 수도 있다. 
-  업데이트된 컨텐츠를 다른 시스템(클라이언트)에 push 할 때 사용하기 좋음

### gRPC
- RPC(Remote Procedure Call)라는 오래된 방식에 대한 새로운 접근
    - RPC는 원격 서버에서 프로 시저를 실행하는 방법으로, 워크 스테이션에서 몇 마일 떨어진 친구의 컴퓨터에서 프로그램을 실행하는 것과 비슷하다.
    - RPC의 단점이 REST 를 개발하게 했음 
- gRPC 와 REST 는 상호작용을 무엇으로 하냐에 차이가 있음
    - gRPC는 **아키텍쳐 그 자체 대신 서버-클라이언트의 관계에 의해** 상호작용이 정의되고 제한됨
        - RPC는 client에게 상호작용의 책임을 부여
        - 많은 처리 및 계산은 리소스를 호스팅하는 원격 서버에게 부여
    - REST는 request 에 표준화된 계약에 의해 상호작용이 이루어짐

- 리소스를 요구하는 REST와 달리 **매우 저전력의 상황**에서 사용할 수 있음.
    - 저전력 장치를 위한 맞춤형 통신이 필요한 IoT 장치 및 기타 솔루션에 사용함
    - 통신 비용을 줄일 수 있음
    - **데이터 요청을위한 빠르고 가벼운 시스템**에 적합
- 오픈소스임
- SSL/TLS를 사용한 강력한 인증 시스템을 가지고 있음

<br>

### 비교
- REST
    - **A stateless architecture** for data transfer that is dependent on **hypermedia**
- GraphQL
    - An approach wherein the **user defines the expected data and format of that data**
- Webhooks
    - Data updates to **be served automatically**, rather than requested.
- gRPC
    - A **nimble and lightweight system** for requesting data

<br>

## 참고자료
---
<a href="https://nordicapis.com/when-to-use-what-rest-graphql-webhooks-grpc/">https://nordicapis.com/when-to-use-what-rest-graphql-webhooks-grpc/</a>