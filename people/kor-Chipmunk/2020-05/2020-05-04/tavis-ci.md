# Travis CI 배포 자동화

## Travis CI 연동하기

Travis CI는 깃허브에서 제공하는 무료 CI 서비스다. 다른 CI 도구로는 젠킨스가 있다. 젠킨스는 설치형이기에 이를 위한 EC2 인스턴스가 하나 더 필요하다. 신생 단계에서 배포를 위한 EC2 인스턴스는 부담스러울 수 있다. 따라서 오픈소스 웹 서비스인 Travis CI를 사용한다.

참고로 AWS에서 Travis CI와 같은 CI 도구로 CodeBuild를 제공한다. 그러나 빌드 시간 만큼 요금이 부과되는 구조라 초기에 사용하기에는 부담스럽다. 실제 서비스되는 EC2, RDS, S3 외에는 비용 부분을 최소화하는 것이 좋다.

### Travis CI 웹 서비스 설정

[https://tavis.ci.org/]() 에서 깃허브 계정으로 로그인을 한 뒤 Settings 메뉴로 들어간다.

깃허브 저장소 검색창에 저장소 이름을 입력해 찾는다. 그 다음 오른쪽 상태바를 활성화 시킨다. 활성화된 저장소를 클릭해 저장소 빌드 히스토리 페이지로 이동한다.

Travis CI 웹사이트에서 설정은 이것으로 끝이다. 상세한 설정은 프로젝트의 `yml` 파일로 진행한다.

### 프로젝트 설정

Travis CI의 상세한 설정은 프로젝트에 존재하는 `.travis.yml` 파일이다. yml 파일 확장자를 YAML(야믈) 이라고 한다.

YAML은 쉽게 말해 JSON에서 괄호를 제거한 것이다. 현재 많은 프로젝트와 서비스들이 이 YAML을 적극적으로 사용 중이다. Travis CI 역시 설정을 YAML 으로 하고 있다.

프로젝트의 `build.gradle` 파일과 같은 위치에서 `.travis.yml` 파일을 생성해 다음 코드를 추가한다.

```yml
language: java
jdk:
  - openjdk8

branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - rhj4862@gmail.com
```

1. `branches`
   1. Travis CI를 어느 브랜치치가 푸시될 때 수행할지 지정한다.
   2. 현재 옵션은 오직 `master` 브랜치에 푸시될 때 수행한다.
2. `cache`
   1. 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐싱하여, 같은 의존성은 다음 배포 때 부터 다시 받지 않도록 설정한다.
3. `script`
   1. `master` 브랜치에 푸시되었을 때 수행하는 명령어다.
   2. 프로젝트 내부에 둔 `gradlew`을 통해 clean & build 작업을 수행한다.
4. `notifications`
   1. Travis CI 실행 완료 시 자동으로 알람이 가도록 설정한다.

`master` 브랜치에 커밋과 푸시를 하고 다시 Travis CI 저장소 페이지를 확인한다.

빌드가 성공했다면, .travis.yml에 등록한 이메일로 알람이 간다.

### 권한 오류

`gradlew` 권한 오류가 아래와 같이 발생했다면, 권한을 추가하는 작업을 `.travis.yml` 파일에 추가해야 한다.

```yml
...
before_install:
    - chmod +x gradlew
...
```

# 참고도서

스프링 부트와 혼자 구현하는 웹 서비스