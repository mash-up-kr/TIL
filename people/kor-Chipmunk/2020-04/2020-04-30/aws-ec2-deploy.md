# EC2 서버에 프로젝트를 배포해 보자

## EC2에 프로젝트 Clone 받기

EC2에 접속해 깃을 설치한다.

```sh
# git 패키지 설치
sudo yum install git

# git 버전 보기
git --version

# git clone으로 프로젝트를 저장할 디렉토리를 생성
mkdir ~/app && mkdir ~/app/step1

# 생성된 디렉토리로 이동
cd ~/app/step1

# 내 프로젝트 레포를 클론한다.
git clone 복사한 주소
```

코드들이 잘 실행되는지 테스트로 확인한다.

```sh
# gradlew 권한 변경
chmod +x ./gradlew

# gradlew 테스트 실행
./gradlew test

# Success case
# BUILD SUCCESSFUL in 3s
# 5 actionable tasks: 5 up-to-date
# ...
```

다음과 같은 스크립트 코드를 추가한다.

```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=freelec-springboot2-webservice

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar $REPOSITORY/$PROJECT_NAME/build/libs/$JAR_NAME 2>&1 &
```

`deploy.sh` 파일명으로 저장하고 실행 권한을 추가한다.

```sh
# 실행 권한 추가
chmod +x ./deploy.sh

# deploh.sh 실행
./deploy.sh
```

각 스크립트 구문 설명은 다음과 같다.

1. `REPOSITORY=/home/ec2-user/app/step1`
   1. 프로젝트 디렉토리 주소는 스크립트 내에서 자주 사용하는 값이므로 변수로 저장
   2. PROJECT_NAME 또한 변수로 저장
   3. 쉘에서는 타입 없이 선언하여 저장
   4. 쉘에서는 $변수명으로 변수를 사용
2. `cd $REPOSITORY/$PROJECT_NAME`
   1. 제일 처음 git clone 받은 디렉토리로 이동
   2. 쉘 변수 값에 따라 디렉토리를 이동
3. `git pull`
   1. 디렉토리 이동 후, master 브랜치의 최신 내용을 받음
4. `./gradlew build`
   1. 프로젝트 내부의 gradlew로 build를 수행
5. `cp ./build/libs/*.jar $REPOSITORY/`
   1. build의 결과물인 jar 파일을 복사해 jar 파일을 모아둔 위치로 복사
6. `CURRENT_PID=$(pgrep -f springboot-webservice)`
   1. 기존에 수행 중이던 스프링 부트 애플리케이션을 종료
   2. `pgrep` 은 process id만 추출하는 명령어
   3. `-f` 옵션은 프로세스 이름으로 탐색
7. `if ~ else ~ fi`
   1. 현재 구동중인 프로세스가 있는지 없는지를 확인
   2. `[ -z "$CURRENT_PID" ]`
8. `JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)`
   1. 새로 실행할 jar 파일명을 탐색
   2. `tail -n` 으로 가장 나중의 jar 파일 (최신 파일)을 변수에 저장
9. `nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &`
   1.  찾은 jar 파일명으로 해당 jar 파일을 `nohup` 으로 실행
   2.  스프링 부트의 장점으로 특별히 외장 톰캣을 설치할 필요 없음
   3.  내장 톰캣을 사용해 jar 파일만 있으면 바로 웹 애플리케이션 서버를 실행할 수 있음
   4.  일반적으로 자바를 실행할 때 `java -jar` 명령어를 사용
       1.  그러나 사용자가 터미널 접속을 끊을 때 애플리케이션도 같이 종료
   5.  애플리케이션을 계속 구동될 수 있도록 `nohup` 명령어를 사용

## 외부 Security 파일 등록하기

Secret 키가 담긴 파일을 깃허브에 제외시켰기에 애플리케이션 실행에 오류가 발생한다.  
서버 환경으로 키를 옮기자.

```sh
vim /home/ec2-user/app/application-oauth.properties
```

`app` 디렉토리에 `application-oauth.properties` 파일을 생성하여 키값을 넣는다.

`deploy.sh` 파일에서 jar 파일을 실행할 때, 설정값을 `app` 디렉토리의 설정파일에 연결할 수 있도록 수정한다.

```sh
.
.
.

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
    $REPOSITORY/$JAR_NAME 2>&1 &
```

1. `-Dspring.config.location`
   1. 스프링 설정 파일 위치를 설정한다.
   2. 기본 옵션들을 담고 있는 `application.properties`과 OAuth 설정을 담고있는 `application-oauth.properties`의 위치를 지정한다.
   3. `classpath` 가 붙으면 jar 안에 있는 `resources` 디렉토리를 기준으로 경로가 생성된다.
   4. `application-oauth.properties` 은 외부에 파일이 있기 때문에 절대경로를 사용한다.

# 참고 도서

스프링 부트와 AWS로 혼자 구현하는 웹 서비스