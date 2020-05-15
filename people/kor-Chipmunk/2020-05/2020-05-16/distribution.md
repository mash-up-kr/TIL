# 무중단 배포

## 무중단 배포 스크립트 만들기

스크립트 작업 전, API를 하나 추가해야 한다. 배포 시에 8081을 쓸지, 8082를 쓸지 판단하는 기준이 된다.

### profile API 추가

`src/.../web/ProfileController`를 만들어 다음과 같이 간단한 API 코드를 추가한다.

```java
@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");
        String defaultProfile = profiles.isEmpty() ? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
}
```

1. `env.getActiveProfiles()`
   1. 현재 실행 중인 ActiveProfile을 모두 가져온다.
   2. 즉, real, oauth, real-db 등이 활성화되어 있다면 3개가 모두 담겨있다.
   3. 여기서 real, real1, real2는 모두 배포에 사용될 profile이라 이 중 하나라도 있으면 그 값을 반환하도록 한다.
   4. step2의 real도 감안한다.

코드가 잘 작동하는지 테스트 코드를 작성한다. 위 컨트롤러는 스프링 환경이 필요하지 않다. `@SpringBootTest` 없이 테스트 코드를 작성할 수 있다.

`test/.../web/ProfileControllerUnitTest` 유닛 테스트 파일을 작성한다.

```java
import org.junit.Test;
import org.springframework.mock.env.MockEnvironment;

import static org.assertj.core.api.Assertions.assertThat;

public class ProfileControllerUnitTest {

    @Test
    public void real_profile이_조회된다() {
        //given
        String expectedProfiile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfiile);
        env.addActiveProfile("oauth");
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfiile);
    }

    @Test
    public void real_profile이_없으면_첫번째가_조회된다() {
        //given
        String expectedProfile = "oauth";
        MockEnvironment env = new MockEnvironment();

        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void active_profile이_없으면_default가_조회된다() {
        //given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();
        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

}
```

`ProfileController`와 `Environment` 모두 자바 클래스(인터페이스)이기 때문에 쉽게 테스트할 수 있다. `Environment`는 인터페이스라 가짜 구현체인 `MockEnvironment`(스프링에서 제공)를 사용해 테스트하면 된다.

생성자 DI가 얼마나 유용한지 확인할 수 있다. 만약 `Environment`를 `@Autowired`로 DI 받았다면, 항상 스프링 테스트를 해야만 했다.

`/profile`이 인증 없이 호출이 될 수 있도록 `config/auth/SecurityConfig` 클래스에 제외 코드를 추가한다.

```java
.antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
```

`SecurityConfig` 설정이 잘 되었는지 테스트 코드로 검증한다. 스프링 시큐리티 설정을 불러와야 하니 `@SpringBootTest`를 사용하는 테스트 클래스(`ProfileControllerTest`)를 하나 더 추가한다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ProfileControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void profile은_인증없이_호출된다() throws Exception {
        String expected = "default";

        ResponseEntity<String> response = restTemplate.getForEntity("/profile", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isEqualTo(expected);
    }

}
```

깃허브로 푸시해 배포한다. 배포가 끝나면 브라우저에 `/profile`에 접속해 profile이 잘 나오는지 확인한다.

### real1, real2 profile 생성

현재 EC2 환경에서 실행되는 profile은 real밖에 없다. 해당 profile은 Travis CI 배포 자동화를 위한 profile이니 무중단 배포를 위한 profile 두 개(real1, real2)를 추가한다.

`application-real1.properties` 파일은 다음과 같다.

```properties
server.port=8081
spring.profiles.include=oauth,real-db
spring.jpa.properties.hinernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```

`application-real2.properties` 파일은 다음과 같다.

```properties
server.port=8082
spring.profiles.include=oauth,real-db
spring.jpa.properties.hinernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```

real 프로파일과 다른 점은 포트 번호다. 8080이 아닌 8081/8082로 되어 있다. 깃허브로 푸시해 저장한다.

### 엔진엑스 설정 수정

무중단 배포의 핵심은 엔진엑스 설정이다. 배포 때마다 엔진엑스의 프록시 설정(스프링 부트로 요청을 보내는)이 순식간에 교체된다. 프록시 설정이 교체될 수 있도록 설정을 추가한다.

엔진엑스 설정이 모여있는 `/etc/nginx/conf.d/` 에 `service-url.inc`라는 파일을 생성한다.

```sh
sudo vim /etc/nginx/conf.d/service-url.inc
```

그리고 다음 코드를 입력한다.

```sh
set $service_url http://127.0.0.1:8080;
```

엔진엑스가 사용할 수 있게 설정한다. `nginx.conf` 파일을 연다.

```sh
sudo vim /etc/nginx/nginx.conf
```

`location /` 부분을 찾아 다음과 같이 변경한다.

```sh
include /etc/nginx/conf.d/service-url.inc;

location / {
    proxy_pass $service_url;
    ...
}
```

저장한 뒤 엔진엑스를 재시작한다.

```sh
sudo service nginx restart
```

### 배포 스크립트들 작성

먼저 step2와 중복되지 않기 위해 EC2에 step3 디렉토리를 생성한다.

```sh
mkdir ~/app/step3 && mkdir ~/app/step3/zip
```

무중단 배포는 앞으로 step3를 사용한다. `appspec.yml` 역시 step3로 배포되도록 수정한다.

```yml
version: 0.0
os: linux
files:
  - source:  /
    destination: /home/ec2-user/app/step3/zip/
    overwrite: yes
...
```

무중단 배포를 진행할 스크립트들은 총 5개다.

1. `stop.sh` : 기존 엔진엑스에 연결되어 있진 않지만, 실행 중이던 스프링 부트를 종료한다.
2. `start.sh` : 배포할 신규 버전 스프링 부트 프로젝트를 stop.sh로 종료한 `profile`로 실행한다.
3. `health.sh` : `start.sh` 로 실행시킨 프로젝트가 정상적으로 실행됐는지 체크한다.
4. `switch.sh` : 엔진엑스가 바라보는 스프링 부트를 최신 버전으로 변경한다.
5. `profile.sh` : 앞선 4개 스크립트 파일에서 공용으로 사용할 `profile`과 포트 체크 로직을 담당한다.

`appsec.yml` 파일에서 앞언 스크립트를 사용하도록 설정한다.

```yml
...

hooks:
  AfterInstall:
    - location: stop.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트를 종료합니다.
      timeout: 60
      runas: ec2-user
  ApplicationStart:
    - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작합니다.
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh # 새 스프링 부트가 정상적으로 실행됐는지 확인한다.
      timeout: 60
      runas: ec2-user
```

Jar 파일이 복사된 이후부터 차례로 앞선 스크립트들이 실행된다. 이 스크립트들 역시 `scripts/` 디렉토리에 추가한다.

#### `profile.sh`
```sh
#!/usr/bin/env bash

# bash는 return value가 안되니 *제일 마지막줄에 echo로 해서 결과 출력*후, 클라이언트에서 값을 사용한다

# 쉬고 있는 profile 찾기: real1이 사용중이면 real2가 쉬고 있고, 반대면 real1이 쉬고 있음
function find_idle_profile()
{
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi

    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}"
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
    IDLE_PROFILE=$(find_idle_profile)

    if [ ${IDLE_PROFILE} == real1 ]
    then
      echo "8081"
    else
      echo "8082"
    fi
}
```

1. `$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)`
   1. 현재 엔진엑스가 바라보고 있는 스프링 부트가 정상적으로 수행 중인지 확인한다.
   2. 응답값을 HttpStatus로 받는다.
   3. 정상이면 200, 오류가 발생한다면 400~503 사이로 발생하니 400 이상은 모두 예외로 보고 real2를 현재 profile로 사용한다.
2. `IDLE_PROFILE`
   1. 엔진엑스와 연결되지 않은 profile이다.
   2. 스프링 부트 프로젝트를 이 profile로 연결하기 위해 반환한다.
3. `echo "${IDLE_PROFILE}"`
   1. bash라는 스크립트는 값을 반환하는 기능이 없다.
   2. 따라서 제일 마지막 줄에 echo로 결과를 출력 후, 클라이언트에서 그 값을 잡아서 ($(find_idle_profile)) 사용한다.
   3. 중간에 echo를 사용해선 안 된다.

#### `stop.sh`
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동 중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```

1. `ABSDIR=$(dirname $ABSPATH)`
   1. 현재 stop.sh가 속해 있는 경로를 찾는다.
   2. 하단의 코드와 같이 `profile.sh`의 경로를 찾기 위해 사용된다.
2. `source ${ABSDIR}/profile.sh`
   1. 자바로 보면 일종의 import 구문이다.
   2. 해당 코드로 인해 `stop.sh` 에서도 `profile.sh`의 여러 function을 사용할 수 있게 된다.

#### `start.sh`
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=springboot-example

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."
nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

기본적인 스크립트는 step2의 `deploy.sh`와 유사하다.  
다른 점은 `IDLE_PROFILE`을 통해 `properties` 파일들을 가져오고 (application-$IDLE_PROFILE.properties), active profile을 지정하는 것(-Dspring.profiles.active=$IDLE_PROFILE) 뿐이다.  
여기서도 `IDLE_PROFILE`을 사용하니 `profile.sh`을 가져와야 한다.

#### `health.sh`
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      switch_proxy
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

엔진엑스와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지를 체크한다.  
잘 떴는지 확인되어야 엔진엑스 프록시 설정을 변경(switch_proxy) 합니다.  
엔진엑스 프록시 설정 변경은 switch.sh에서 수행한다.

#### `switch.sh`
```sh
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc

    echo "> 엔진엑스 Reload"
    sudo service nginx reload
}
```

1. `echo "set \$service_url http://127.0.0.1:${IDLE_PORT};"`
   1. 하나의 문장을 만들어 파이프라인(|)으로 넘겨주기 위해 echo를 사용한다.
   2. 엔진엑스가 변경할 프록시 주소를 생성한다.
   3. 쌍따음표(")을 사용해야 한다.
   4. 사용하지 않으면 $service_url을 그대로 인식하지 못하고 변수를 찾게 된다.
2. `| sudo tee /etc/nginx/conf.d/service-url.inc`
   1. 앞에서 넘겨준 문장을 `service-url.inc`에 덮어쓴다.
3. `sudo service nginx reload`
   1. 엔진엑스 설정을 다시 불러온다.
   2. `restart`와는 다르다.
   3. `restart`는 잠시 끊기는 현상이 있지만, `reload`는 끊김 없이 다시 불러온다.
   4. 다만, 중요한 설정들은 반영되지 않으므로 그럴 경우 `restart` 해야 한다.
   5. 여기선 외부의 설정 파일인 `service-url`을 다시 불러오는 거라 `reload`로 가능하다.

### 무중단 배포 테스트

잦은 배포로 Jar 파일명이 겹칠 수 있다. 매번 버전을 올리는 것이 귀찮으므로 자동으로 버전값이 변경될 수 있도록 Version 뒤에 DateTime을 추가할 수 있다.

`build.gradle`에 version 속성을 다음과 같이 변경한다.

```gradle
version '1.0.8-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
```

`build.gradle`은 `Groovy` 기반의 빌드툴이기에, `Groovy` 언어의 여러 문법들을 사용할 수 있다. `new Date()`로 빌드할 때마다 그 시간이 버전에 추가하도록 구성했다.

깃허브로 푸쉬후 배포가 자동으로 진행되면 `CodeDeploy` 로그로 잘 진행되는지 확인해본다.

```sh
tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.lgo
```

스프링 부트 로그도 보고 싶다면 다음 명령어로 확인할 수 있다.

```sh
vim ~/app/step3/nohup.out
```

자바 애플리케이션 실행 여부는 다음 명령어로 확인할 수 있다.

```sh
ps -ef | grep java
```

## 참고 도서

스프링 부트와 혼자 구현하는 웹 서비스