# AWS RDS

AWS에서 모니터링, 알람, 백업, HA 구성 작업을 지원하는 관리형 서비스 RDS(Relational Database Service)를 제공한다. 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같이 잦은 운영 작업을 자동화했다. 용량도 유연하게 조절이 가능해 예상치 못한 상황에 대비할 수 있다.

* HA 구성 : 고가용성 솔루션, High Availability, 서비스의 연속성을 유지하는 방법 또는 솔루션을 뜻한다. 장애 발생 후 수 분 내 서비스 재개를 목표로 운영용 서버 2대를 Active 또는 Stand By 형태로 구성하고 데이터를 실시간으로 복제한다. Active 서버가 장애 시 장애 대기 서버(Stand By)로 서비스를 재개한다. 다만, 데이터 실시간 복제는 거리상의 제약이 있을 수 있으므로 LAN (내부 네트워크) 상에서 이루어진다.

DB 엔진 유형은 다음과 같다.
- Amazon Aurora
  - 고성능 상용 데이터베이스의 성능과 가용성에 오픈 소스 데이터베이스의 간편성과 비용 효율성을 결합하였으며 클라우드를 위해 구축된 MySQL 및 PostgreSQL 호환 관계형 데이터베이스
  - 표준 MySQL 데이터베이스보다 최대 5배 빠르고 표준 PostgreSQL 데이터베이스보다 3배 빠르다.
  - 1/10의 비용으로 상용 데이터베이스의 보안, 가용성 및 안정성을 제공
  - 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같은 시간 소모적인 관리 작업을 자동화하는 Amazon Relational Database Service(RDS)에서 Amazon Aurora의 모든 것을 관리
  - 내결함성을 갖춘 자가 복구 분산 스토리지 시스템으로, 데이터베이스 인스턴스당 최대 64TB까지 자동으로 확장
- MySQL
- MariaDB
- PostgreSQL
- Oracle
- Microsoft SQL Server

다른 데이터베이스를 선택해야 할 이유가 있는 것이 아니라면 MySQL, MariaDB, PostgreSQL 중에 고르는 걸 추천  
MariaDB를 추천하는 이유는 다음과 같다.

- 가격
- Amazon Aurora(오로라) 교체 용이성

- RDS의 가격은 라이센스 비용에 영향을 받는다. 상용 데이터베이스인 Oracle, MSSql이 오픈 소스 라이브러리인 MySQL, MariaDB, PostgreSQL 보다 동일한 사양 대비 가격이 더 높다.
- 클라우드 서비스에 가장 적합한 데이터베이스이기에 많은 회사들이 Amazon Aurora 를 선택한다.
- 단, 프리티어 대상이 아니며 최저 비용이 월 10만원 이상이다.

가장 인기있는 오픈소스 데이터베이스는 MySQL 이다. **단순 쿼리 처리 성능**이 어떤 제품보다 압도적이고 오래 사용되어 왔기에 성능과 신뢰성 부분에서 꾸준히 개선되어 온 것이 장점이다. MariaDB는 MySQL을 기반으로 만들어져 쿼리 사용법은 MySQL과 유사하다. MariaDB가 MySQL 보다 좋은 이유는 다음과 같다.
- 동일 하드웨어 사양으로 MySQL보다 향상된 성능
- 좀 더 활성화된 커뮤니티
  - MariaDB 개발이 더 개방적임
  - 공개 메일링 리스트와 버그트래커가 있고 완전히 공개된 리포가 있음
  - MariaDB가 github 컨트리뷰터 수가 많음
  - 문서화에 더 열심
- 좀 더 다듬고 더 다양한 기능
  - MySQL에 비해 더 다양한 기능을 지원
  - MariaDB는 Dynamic 칼럼 지원 등
- 다양한 스토리지 엔진
  - Connect 와 Cassandra, 샤딩을 위한 Spider, 프랙탈 인덱스의 TokuDB
  - MySQL 에서도 써드파티 플러그인 형태로 사용이 가능하나, MariaDB는 공식 릴리즈에 포함시켜 사용
- 빠르고 투명한 보안패치 릴리즈
  - 오라클은 3개월 주기로 보안패치를 적용 (MySQL 은 2개월 주기)
  - 간혹 보안 정보와 업그레이드가 싱크가 맞지 않는 경우가 발생함
  - MySQL 은 릴리즈노트에 CVE 식별번호가 모두 리스트업 되지 않음
  - 이슈와 픽스에 대한 확인이 모호하여 불만들이 많음
    - https://lists.launchpad.net/maria-discuss/msg00514.html
  - 백포팅이 불가능하게 함, 최신의 MySQL 릴리즈로 업그레이드 할 수 밖에 없는 상황이 있음
  - MariaDB 는 CVE 식별번호를 명시하고 이슈 관리를 잘함

참고 링크 : https://xdhyix.wordpress.com/2016/03/24/mysql-%EC%97%90%EC%84%9C-mariadb-%EB%A1%9C-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-%ED%95%B4%EC%95%BC%ED%95%A0-10%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0/

## RDS 운영환경에 맞는 파라미터 설정하기

RDS를 처음 생성하면 몇 가지 설정을 필수로 해야 한다.

1. 타임존
2. Character Set
3. Max Connection

파라미터 그룹 탭으로 이동한다. 파라미터 그룹 생성 버튼으로 파라미터 그룹 세부 정보를 입력할 수 있는 창이 나타난다. 생성한 MariaDB와 같은 버전을 맞춘다.  
해당 파리미터 그룹을 클릭해 파라미터 편집 버튼으로 편집 모드로 들어간다. time_zone 을 검색해 **Asia/Seoul** 으로 변경한다.

다음으로 Character Set을 변경한다. 아래 8개 항목 중 character 항목들은 모두 utf8mb4로, collation 항목들은 utf8mb4_general_ci로 변경한다. utf8과 utf8mb4의 차이는 이모지 저장 가능 여부다.

- character_set_client
- character_set_connection
- character_set_database
- character_set_filesystem
- character_set_results
- collaction_connection
- collation_server

Max Connection은 인스턴스 사양에 따라 자동으로 정해진다. 프리티어 사양으로 약 60개의 커넥션만 가능하다. 넉넉한 값인 150으로 설정한다.  
RDS 사양이 높아지면 기본값으로 다시 돌려놓는다.

생성된 파라미터 그룹을 데이터베이스에 연결한다. 데이터베이스 항목을 수정해 DB 파라미터 그룹을 수정한다.

## 내 PC에서 RDS 접속하기

데이터베이스 항목을 선택해 세부 정보 항목으로 들어간다. VPC 보안 그룹을 클릭한다. EC2에 사용된 보안 그룹의 그룹 ID를 복사한다.  
복사된 보안 그룹 ID와 내 IP를 RDS 보안 그룹의 인바운드로 추가한다. 규칙 유형은 MYSQL/Aurora를 선택하면 3306 포트가 선택된다.

## Database 플러그인 설치

로컬에서 원격 데이터베이스에 접속할 때 GUI 클라이언트를 많이 사용한다. MySQL의 대표적인 클라이언트로 Workbench, SQLyog(유료), Sequel Pro(맥 전용), DataGrip(유료) 등이 있다. 인텔리제이에서도 Database 플러그인이 있다.

# 참고 도서

스프링 부트와 AWS로 혼자 구현하는 웹 서비스