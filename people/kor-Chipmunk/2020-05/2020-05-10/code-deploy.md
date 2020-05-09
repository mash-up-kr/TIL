# Travis CI 배포 자동화

## CodeDeploy 에이전트 설치

EC2에 접속해 다음 명령어를 입력한다.

```sh
aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2
```

성공했다면 아래 메시지가 나타난다. `download: s3://aws-codedeploy-ap-northeast-2/latest/install to ./install`

install 파일에 실행 권한을 추가한다.

```sh
chmod +x ./install
```

install 파일로 설치를 진행한다.

```sh
sudo ./install auto
```

설치가 끝났으면 Agent가 정상적으로 실행되고 있는지 상태 검사를 한다.

```sh
sudo service codedeploy-agent status
```

다음 메시지가 나타나면 정상이다.

```
The AWS CodeDeploy agent is running as PID xxx
```

만약 설치 중 다음 메시지가 나타난다면 루비라는 언어가 설치가 안 되었기 때문이다. `/usr/bin/env: ruby: No such file or direcotry`

이럴 경우 `yum install` 로 루비를 설치한다.

```sh
sudo yum install ruby
```

## CodeDeploy를 위한 권한 생성

CodeDeploy에서 EC2에 접근하려면 마찬가지로 권한이 필요하다. AWS의 서비스의 IAM 역할을 생성한다. AWS 서비스의 CodeDeploy를 차례로 선택한다.

권한이 하나 뿐이기에 바로 진행한다.

Name 키에 원하는 이름으로 태그를 만든다.

역할 이름과 설정 항목들을 검토한 뒤 생성 완료한다.

## CodeDeploy 생성

AWS의 대표적인 배포 서비스 세 가지의 이름과 특징은 다음과 같다.

- Code Commit
  - 깃허브와 같은 코드 저장소의 역할
  - 프라이빗 기능을 지원한다는 강점
  - 그러나 깃허브에서 무료로 프라이빗 지원을 하고 있어서 거의 사용하고 있지 않다.
- Code Build
  - Travis CI와 마찬가지로 빌드용 서비스
  - 멀티 모듈을 배포해야 하는 경우 사용해 볼만 하다.
  - 규모가 있는 서비스에서는 대부분 젠킨스/팀시티 등을 이용해 사용할 일이 없다.
- CodeDeploy
  - AWS의 배포 서비스
  - CodeDeploy는 앞서 소개한 서비스와는 달리 대체재가 없다.
  - 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포 등 많은 기능을 지원

CodeDeploy 서비스 페이지에 들어가 애플리케이션 생성 버튼을 누른다.

애플리케이션 이름과 컴퓨팅 플랫폼을 `EC2/온프레미스` 을 선택한다.

배포 그룹 생성 버튼을 클릭해 페이지를 이동한다.

**배포 그룹 이름**과 **서비스 역할**을 등록한다. **서비스 역할**은 방금 생성한 CodeDeploy용 IAM 역할을 선택한다.

**배포 유형**은 **현재 위치를** 선택한다. 배포할 서비스가 2대 이상이라면 블**루/그린**을 선택한다. 1대의 EC2에만 배포하므로 선택하지 않는다.

환경 구성에서는 **Amazon EC2 인스턴스**에 체크한다.

마지막으로 배포 구성은 **CodeDeployDefault.AllAtOnce** 을 선택한다. 로드밸런싱은 체크 해제한다.

배포 구성은 한번 배포할 때 몇 대의 서버에 배포할지를 결정한다. 2대 이상이라면 1대씩 배포할지, 30% 혹은 50%로 나눠서 배포할지 등등 여러 옵션을 선택한다. 1대 서버는 전체 배포하는 옵션으로 선택하면 된다.

Travis CI와 CodeDeploy를 연동하는 일만 남았다.

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스