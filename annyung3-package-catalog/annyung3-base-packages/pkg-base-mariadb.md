# mariadb

## Description:

MariaDB 클라이언트 프로그램과 공유 라이브러리

## Changes on AnNyung:

1. 안녕 리눅스 3.5 에서 10.3 branch 로 업데이트 되었습니다. 1.1 update 및 restart 를 한 후, mysql\_upgrade 를 실행해 주십시오. 1.2 10.1 의 경우에는 10.2 로 업데이트를 한 후에 10.3 으로 순서대로 업데이트를 하십시오.
2. 5.5 및 10.1 compat library를 포함하고 있어 기존의 패키지 의존성 유지가 가능 합니다.

## Sub packages:

* **mariadb-bench** - MariaDB 벤치마크 스크립트와 데이터
* **mariadb-devel** - MariaDB/MySQL 응용 프로그램 개발을 위한 파일들
* **mariadb-embedded** - MariaDB 임베디드 라이브러리
* **mariadb-embedded-devel** - MariaDB 임베디드 라이브러리를 위한 개발 파일들
* **mariadb-garbd** - Galera Arbitrator daemon
* **mariadb-libs** - MariaDB/MySQL clients에 필요한 공유 라이브러리
* **mariadb-server** - MariaDB 서버와 관련 파일들
* **mariadb-test** - MariaDB에서 배포하는 테스트 도구
* [**mariadb-aes256**](../annyung3-core-packages/pkg-core-mariadb-aes256.md) - AES256\_ENCRYPT / AES256\_DECRYPT User define function

