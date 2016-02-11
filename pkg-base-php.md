# php

### Description:
Apache2 PHP 엔진 (mod_php7, libphp7.so)

### Changes on AnNyung:
1. php 7 업데이트
2. exec_dir 기능
3. disable_functions 기본 적용
4. file upload시 image header의 injection code 여부 검사 기능 추가
5. allow_url_fopen과 allow_url_include를 PHP_INI_ALL로 수정
6. allow_include 기능 추가
7. short_open_tag 기본값 Off

### Sub packages:
* **php-cli** - php7 cli 인터페이스
* **php-dba** - PHP7 dba 확장
* **php-dblib** - PHP7 dba 확장
* **php-devel** - php7 확장 개발을 위한 파일들
* **php-extension** - php7 shared extension
* **php-fpm** - php7 fpm engine
* **php-oci** - PHP7 oci8/pdo_oci 확장
* **php-odbc** - PHP7 odbc, pdo_odbc 확장
* **php-pear** - PHP 확장및 응용 프로그램 저장소 프레임웍
* **php-pgsql** - PHP7 pgsql, pdo_pgsql 확장
* **php-recode** - PHP7 recode 확장