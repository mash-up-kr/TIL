# AWS EC2 - 3

## 아마존 리눅스 1 서버 생성 시 꼭 해야 할 설정들

자바 기반의 웹 애플리케이션(톰캣과 스프링부트)을 돌아가도록 필수로 설정해야 한다.

- Java 8 설치
- 타임존 변경 : 기본 서버의 시간은 미국 시간대다. 한국 시간대가 되어야만 우리가 사용하는 시간이 모두 한국 시간으로 등록되고 사용된다.
- 호스트네임 변경 : 현재 접속한 서버의 별명을 등록한다. IP만으로 어떤 서버가 어떤 역할을 하는 지 알 수 없다. 이를 구분하기 위해 호스트 네임을 필수로 등록한다.

### Java 8 설치

아마존 리눅스 1의 경우 기본 자바 버전이 7이다. 자바8을 EC2에 설치하는 방법은 다음과 같다.

```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```

인스턴스의 Java 버전을 8로 변경한다.

```
sudo /usr/sbin/alternatives --config java

# 2 입력
```

버전이 변경되었으면 사용하지 않는 Java7을 삭제한다.

```
sudo yum remove java-1.7.0-openjdk
```

현재 버전이 Java8 인지 확인한다.

```
java -version
```

### 타임존 변경

```
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime

date
# 타임존 KST로 변경된 것 확인
```

### HostName 변경

```
sudo vim /etc/sysconfig/network
```

에서 HOSTNAME 값을 원하는 서비스명으로 변경한다.

```
sudo reboot
```

변경한 후 서버를 재부팅한다.

호스트 네임이 변경되면 한 가지 작업을 더 해야 한다.

호스트 주소를 찾을 때 가장 먼저 검색해보는 /etc/hosts 에 변경한 hostname 을 등록한다.

```
sudo vim /etc/hosts

127.0.0.1 등록한 HOSTNAME
```

다음 명령어로 정상적으로 등록되었는지 확인할 수 있다.

```
curl 등록한 호스트 네임

# 등록 실패한 경우
# Could not resolve host

# 등록 성공한 경우
# Failed to connect to
```

Failed to connect to 메시지는 아직 80 포트로 실행된 서비스가 없음을 의미합니다. 즉, curl 호스트 이름으로 실행은 잘 되었음을 뜻한다.

# 참고 도서

스프링 부트와 AWS로 혼자 구현하는 웹 서비스