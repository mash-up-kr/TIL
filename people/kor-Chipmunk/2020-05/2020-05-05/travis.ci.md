# Travis CI 배포 자동화

## Travis CI와 AWS S3 연동하기

S3는 AWS에서 제공하는 일종의 파일 서버다. 이미지 파일을 비롯해 정적 파일들을 관리하거나 배포 파일들을 관리 하는 용도로 쓴다.

보통 이미지 업로드를 구현한다면 S3를 이용해 구현하기도 한다. S3와 Travis CI를 연동한다면 다음의 구조를 가진다.

1. Github -> Travis CI
2. Travis CI -> AWS S3 : jar 전달
3. Travis CI -> AWS CodeDeploy : 배포 요청
4. AWS S3 -> AWS CodeDeploy : jar 전달
5. AWS CodeDeploy -> AWS EC2 : 배포
6. AWS EC2 -> Spring Boot

실제 배포는 AWS CodeDeploy 서비스를 이용한다. S3 연동이 필요한 이유는 Jar 파일을 전달하기 위함이다. CodeDeploy는 저장 기능이 없다. Travis CI가 빌드한 결과물을 받아 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요하다. 이럴 때 AWS S3를 사용한다.

CodeDeploy가 빌드도 하고 배포도 할 수 있다. 깃허브 코드롤 가져오는 기능을 지원한다. 그러나 빌드 없이 배포만 필요할 때 대응하기 어렵다.

빌드와 배포가 분리된다면 예전에 만들어진 Jar를 재사용하면 된다. 그러나 CodeDeploy는 항상 빌드를 먼저 해 확장성이 떨어진다. 웬만하면 빌드와 배포를 분리하는 것이 좋다.

### AWS Key 발급

보통 AWS 서비스에 외부 서비스가 접근할 수 없다. 접근 가능한 권한을 가진 Key를 생성해 사용해야 한다. AWS에서 이러한 인증과 관련한 기능을 제공하는 서비스가 이싿. 바로 IAM(Identity and Access Management) 이다.

IAM은 AWS에서 제공하는 서비스의 접근 방식과 권한을 관리한다. IAM을 통해 Travis CI가 AWS의 S3와 CodeDeploy에 접근할 수 있도록 설정한다. AWS 웹 콘솔에서 IAM을 검색해 이동한다. IAM 페이지 왼쪽 사이드바에서 `사용자 -> 사용자 추가` 버튼을 클릭한다.

사용자의 이름과 액세스 유형을 선택한다. 액세스 유형은 **프로그래밍 방식 액세스**다. 권한 설정 방식은 **기존 정책 직접 연결** 을 선택한다. `s3full`로 검색해 체크하고 다음 권한으로 `CodeDeployFull` 을 검색해 체크한다.

서비스하는 기업에서 권한 또한 S3와 CodeDeploy를 분리해서 관리하기도 한다.

태그는 Name 값을 지정한다. 인지 가능한 정도의 이름으로 만든다.

액세스 키와 시크릿 키를 Travis CI 저장소 페이지의 Settings 페이지에 `Environment Variables`에 등록한다.

등록된 값들은 `.travis.yml` 파일에서 `$AWS_ACCESS_KEY`, `$AWS_SECRET_KEY` 이름으로 사용할 수 있다. 키를 이용해 Jar를 관리할 S3 버킷을 만들 수 있다.

# 참고도서

스프링 부트와 혼자 구현하는 웹 서비스