# Travis CI 배포 자동화

## appsepc.yml 파일 수정

```yml
version: 0.0
os: linux
files:
  - source:  /
    destination: /home/ec2-user/app/step2/zip/
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
```

들여쓰기에 주의한다.

## 스크립트 파일 수정

스크립트 오류(정규식, 프로세스 찾기 오류)가 있어 수정한다.

`app/step1/deploy.sh` 스크립트 파일은 다음과 같이 수정한다.

```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=springboot-example

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

chmod +x ./gradlew

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}.*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
        -Dspring.profiles.active=real \
        $REPOSITORY/$JAR_NAME 2>&1 &
```

프로젝트에 포함된 `scripts/deploy.sh` 스크립트 파일도 수정한다.

```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=springboot-example

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl $PROJECT_NAME | grep jar | awk '{print $1}')

echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

## 업데이트

`build.gradle` 파일과 `index.mustache` 파일에서 다음 부분을 수정해 깃허브로 푸쉬해보자.

```gradle
version '1.0.1-SNAPSHOT'
```

```html
<h1>스프링 부트로 시작하는 웹 서비스 개정판 2</h1>
```

정상적으로 Deploy에 성공하고 새로고침해보면 반영이 되어있다.

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스