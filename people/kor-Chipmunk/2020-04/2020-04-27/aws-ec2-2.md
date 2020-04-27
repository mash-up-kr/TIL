# AWS EC2 - 2

## EC2 인스턴스 생성

AWS EC2 보안그룹 생성 탭에서 다음과 같은 설정이 있다.

SSH 프로토콜에 포트 22번인 경우는 AWS EC2에 터미널로 접속할 때를 이야기 한다. PEM 키가 없으면 접속이 안되니 전체 오픈(0.0.0.0/0, ::/0) 하는 경우가 많다. PEM키가 노출된 경우 바로 보안에 위협이 가해진다. PEM 키 관리와 지정덴 IP에서만 SSH 접속이 가능하도록 설정하는 것이 좋다.

프로젝트의 기본 포트인 8080을 추가한다.

## EIP 할당

AWS의 고정 IP는 Elastic IP(EIP, 탄력적 IP)라고 한다. 탄력적 IP 메뉴에서 탄력적 IP 주소 할당 버튼을 누르면, 탄력적 IP가 발급된다. 그리고 주소 연결을 해 인스턴스와 IP를 선택하면 탄력적 주소 연결 작업이 완료된다.

## Mac OS & Linux

AWS에 SSH 접속을 하려면, 아래와 같은 명령어가 필요하다.
```
ssh -i pem 키 위치 EC2의 탄력적 IP 주소
```

이 작업을 쉽게할 수 있도록 다음과 같은 명령어를 친다.
```
cp pem 키를 내려받은 위치 ~/.ssh/
```

pem 키를 ~/.ssh/ 디렉토리로 옮기면 ssh 실행 시 pem 키 파일을 자동으로 읽어 접속을 진행한다.  
cp 명령어로 복사가 되었다면 권한을 변경한다.

```
chmod 600 ~/.ssh/pem키이름
```

권한을 변경한 다음 ~/.ssh 디렉토리에 config 파일을 생성한다.

```
vim ~/.ssh/config
```

```
# 주석
Host 본인이 원하는 서비스명
   HostName ec2의 탄력적 IP 주소
   User ec2-user
   IdentityFile ~/.ssh/pem키 이름
```

생성된 config 파일은 실행 권한이 필요하다.

```
chmod 700 ~/.ssh/config
```

다음 명령어로 접속이 가능하다.

```
ssh config에 등록한 서비스명
```

## Windows

- putty.exe
- puttygen.exe

파일이 필요하다.

puttyhen.exe 파일을 실행한 뒤 Conversions -> Import Key 로 pem 키를 불러온다. 그다음 Save private key 버튼으로 pem 키를 ppk 파일로 변환한다.  
putty.exe 파일을 실행해 HostName, Port, Connection type 을 설정한다.

- HostName : username@public_Ip 를 등록한다. `ec2-user@탄력적 IP 주소`
- Port : ssh 접속 포트인 22
- Connection type: SSH 선택

그 다음 Connection -> SSH -> Auth 탭으로 들어가 Browse... 버튼으로 ppk 파일을 로드한다.

Session 탭에서 현재 설정들을 저장할 수 있다. Open 버튼을 눌러 AWS EC2 서버에 SSH 프로토콜로 접속한다.

# 참고 도서

스프링 부트와 AWS로 혼자 구현하는 웹 서비스