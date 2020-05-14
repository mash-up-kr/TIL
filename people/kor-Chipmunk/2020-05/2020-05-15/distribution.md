# 무중단 배포

## 엔진엑스 설치와 스프링 부트 연동하기

### 엔진엑스 설치

```sh
# 엔진엑스 설치
sudo yum install nginx

# 엔진엑스 실행
sudo service nginx start

# 실행 성공 메시지
# Starting nginx:                                            [  OK  ]
```

### 보안 그룹 추가

엔진엑스의 포트번호를 보안 그룹에 추가한다. 포트번호는 기본적으로 80 번호다. EC2 -> 보안 그룹 -> EC2 보안 그룹 선택 -> 인바운드 편집으로 이동해 변경한다.

TCP 80 번호를 모든 IP 위치로 추가한다.

### 리다이렉션 주소 추가

8080이 아닌 80포트로 주소가 변경된다. 구글과 네이버 로그인에도 변경된 주소를 등록해야 한다. 기존에 등록된 리디렉션 주소에서 8080 부분을 제거해 추가 등록한다.

퍼블릭 도메인으로 가면 엔진엑스가 실행된 걸 확인할 수 있다.

### 엔진엑스와 스프링 부트 연동

엔진엑스가 현재 실행 중인 스프링 부트 프로젝트를 바라볼 수 있도록 프록시 설정을 해야한다. 엔진엑스 설정 파일을 연다.

```sh
sudo vim /etc/nginx/nginx/conf
```

```
location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
}
```

`proxy_pass` : 엔진엑스로 요청이 오면 http://localhost:8080로 전달한다.

`proxy_set_header XXX` : 실제 요청 데이터를 header의 각 항목에 할당한다. 

`proxy_set_header X-Real-IP $remote_addr;` : Request Header의 X-Real-IP에 요청자의 IP를 저장한다.

저장 후 다시 엔진엑스를 재시작한다.

```sh
sudo service nginx restart
```

퍼블릭 도메인으로 접속해보면 스프링 부트 프로젝트를 프록시하는 것이 확인되었다.

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스