# 네티 환경 설치하기

## 1. 네티 준비

[http://netty.io/] 에서 네티 최신 파일을 다운로드 받아 압축을 해제한다. `jar` 폴더에서 일부를 골라 사용한다. `all-in-one` 폴더 안에 있는 `netty-all-4.1.50.Final.jar` 파일과 `netty-all-4.1.50.Final-sources.jar` 파일을 사용한다. 두 파일은 네티가 지원하는 모든 기능을 담고 있다. 사용하는 IDE의 프로젝트에 등록하면 된다.

## 2. 네티 개발 환경 설정

네티 jar 파일을 내려받아 직접 프로젝트에서 참조하는 방법으로 개발 환경을 설정한다. 메이븐과 같은 서드 파티 프로그램과의 연동도 가능하다.

최신 이클립스를 다운로드 받고 새 프로젝트를 생성한다. `Create new source folder` 아이콘으로 `src/main`과 `src/test` 초기 소스 코드의 경로를 만든다. 그 다음 기존 `src` 폴더는 `Remove from Build Path` 아이콘을 눌러 빌드 경로에서 삭제해준다.

프로젝트 폴더에 `libs` 라는 폴더를 만들고 두 네티 jar 파일을 복사한다. 그 다음 프로젝트를 오른쪽 클릭으로 메뉴를 열어 `Properties > Java Build Path` 를 선택한다. `Libraries` 탭을 클릭한다. `Add Jars` 버튼을 클릭해 `netty-all.4.1.50.Final.jar` 를 선택한 뒤 완료 버튼을 누른다. jar 파일이 생기면, 확장시켜 `Source attachment: (None)` 항목을 클릭해 `netty-all.4.1.50.Final-sources.jar` 파일을 연동한다.

`~-sources.jar` 파일과 연동하면 이클립스에서 제공하는 Content Assist에 네티 API 설명이 출력되게 된다.

프로젝트 설정을 모두 끝마쳤다.

## 참고 도서

자바 네트워크 소녀 네티