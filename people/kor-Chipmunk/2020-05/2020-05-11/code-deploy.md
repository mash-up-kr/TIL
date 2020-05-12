# Travis CI 배포 자동화

## Travis CI, S3, CodeDeploy 연동

S3에서 넘겨줄 zip 파일을 저장할 디렉토리를 생성한다.

```sh
mkdir ~/app/step2 && mkdir ~/app/step/zip
```

Travis CI의 Build가 끝나면 S3에 zip 파일이 전송되고, 이 zip 파일은 /home/ec2-user/app/step2/zip로 복사되어 압축을 해제해야 한다.

AWS CodeDeploy의 설정은 `appsepc.yml` 파일로 저장한다.

```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwrite: yes
```

1. `version: 0.0`
   1. CodeDeploy 버전
   2. 프로젝트 버전이 아니므로 0.0 외에 다른 버전을 사용하면 오류가 발생한다.
2. source
   1. CodeDeploy에서 전달해 준 파일 중 destination으로 이동시킬 대상을 지정
   2. 루트 경로(/)를 지정하면 전체 파일을 뜻한다.
3. destination
   1. source에서 지정된 파일을 받을 위치다.
   2. 이후 Jar를 실행하는 대상은 destination으로 옮긴 파일들로 진행된다.
4. overwrite
   1. 기존 파일들을 덮어쓸지 결정한다.

`.travis.yml` 설정파일에도 deploy 항목에 CodeDeploy 설정을 추가한다.

```yml
deploy:
  ...

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings에 설정된 값
    bucket: chipmunk-springboot-build # s3 버킷
    key: freelec-springboot2-webservice.zip
    bundle_type: zip # 압축 확장자
    application: freelec-springboot2-webservice # 빌드 파일을 압축해서 전달
    deployment_group: freelec-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true
```

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스