# Travis CI 배포 자동화

## Travis CI와 AWS S3, CodeDeploy 연동하기

AWS의 배포 시스템인 CodeDeploy를 이용하려면, EC2가 CodeDeploy에 접속하게 IAM 역할을 하나 생성해야 한다.

## EC2에 IAM 역할 추가하기

IAM의 역할을 만든다. IAM의 사용자와 역할은 다음과 같은 차이가 있다.

- 역할
  - AWS 서비스에만 할당할 수 있는 권한
  - EC2, CodeDeploy, SQS 등
- 사용자
  - AWS 서비스 외에 사용할 수 있는 권한
  - 로컬 PC, IDC 서버 등
  
**AWS 서비스**를 선택하고 EC2 역할을 선택한다. `EC2RoleForA` 를 검색해 `AmazonEC2RoleforAWS-CodeDeploy`를 선택한다.

태그는 원하는 이름으로 짓는다. 역할의 이름을 등록하고 최종 검토한다.

EC2 인스턴스 목록으로 이동한 뒤, 마우스 오른쪽 버튼으로 눌러 인스턴스 설정 -> IAM 역할 연결/바꾸기를 차례로 선택한다. 방금 선택한 역할을 선택한다.

해당 EC2 인스턴스를 재부팅한다. 재부팅해야만 역할이 정상적으로 적용되니 필수로 재부팅해야 한다.

재부팅이 완료되면 CodeDeploy의 요청을 받을 수 있도록 에이전트를 설치한다.

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스