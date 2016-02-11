# php

### Description:
Apache2 PHP 엔진 (mod_php7, libphp7.so)

### Changes on AnNyung:
1. php 7 업데이트
2. exec_dir (**PHP_INI_SYSTEM**) 기능
 * PHP의 shell injection을 **engine level에서 방어**하기 위한 기능
 * 2005년 부터 KLDP와 N사 T사의 core system에 적용되어 검증
 * PHP 5.4 이전의 safe_mode_exec_dir을 safe_mode가 아닌 경우에도 사용할 수 있도록 수정하고 command 치환 parser를 확장
 * 기본값 _/var/lib/php/bin_
    * 안녕의 PHP 에서 system 함수를 사용하려면 사용하려면 command가 /var/lib/php/bin 에 soft link나 복사 되어야 함.
 * system function에 의해서 실행된 command의 경로를 강제로 지정한 값으로 변환
 ```php
 exec_dir = /path/bin
 system('/some/path/command -option cmd_arg');
 => system('/path/bin/command -option cmd_arg'); 치환됨
 ```
 * 치환 가능 범위
 ```bash
 command; command              => /path/bin/command; /path/bin/command
 command $(command)            => /path/bin/command $(/path/bin/command)
 command $(command $(command)) => /path/bin/command $(/path/bin/command $(/path/bin/command))
 command $(command `command`)  => /path/bin/command $(/path/bin/command `/path/bin/command`)
 command `command`             => /path/bin/command `/path/bin/command`
 command | command             => /path/bin/command | /path/bin/command
 command && command            => /path/bin/command && /path/bin/command
 command || command            => /path/bin/command || /path/bin/command
 ```
  * 참조: http://kldp.org/node/45576
  * 적용 functions
    * 내부적으로 php_exec API를 호출하는 function들
    * [system](http://php.net/manual/kr/function.system.php)
    * [exec](http://php.net/manual/kr/function.exec.php)
    * [passthru](http://php.net/manual/kr/function.passthru.php)
    * [popen](http://php.net/manual/kr/function.popen.php)
    * [escapeshellcmd](http://php.net/manual/kr/function.escapeshellcmd.php)
    * [pcntl_exec](http://php.net/manual/kr/function.pcntl-exec.php)
    * [backtick operator](http://php.net/manual/kr/language.operators.execution.php)
 
 
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