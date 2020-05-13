# Travis CI 배포 자동화

## 배포 자동화 구성

배포 환경에서 실행할 `deploy.sh` 파일을 수정한다.

```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=freelec-springboot2-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl freelec-springboot2-webservice | grep jar | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
   echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
   echo "> kill -15 $CURRENT_PID"
   kill -15 $CURRENT_PID
   sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(LS -TR $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
   -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
   -Dspring.profiles.active=real \
   $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

## .travis.yml 파일 수정

현재 프로젝트의 모든 파일을 zip 파일로 만들지만, 실제로 필요한 파일들은 Jar, appspec.yml, 배포를 위한 스크립트들이다. 나머지를 배제해보자.

```yml
before_deploy:
  - mkdir -p before-deploy
  - cp scripts/*.sh before-deploy/
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy * # before-deploy로 이동 후 전체 압축
  - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동 후 deploy 디렉토리 생성
  - mv before-deploy/before-deploy.zip deploy/freelec-springboot2-webservice.zip # deploy로 zip파일 이동
```

Travis CI는 S3로 특정 파일만 업로드가 안된다. 디렉토리 단위로만 업로드할 수 있기 때문에 deploy 디렉토리는 항상 생성한다.

before-deploy에는 zip파일에 포함시킬 파일들을 저장한다. `zip -r` 명령어로 before-deploy 디렉토리 전체 파일을 압축한다.

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스