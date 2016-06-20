# php

### Description:
Apache2 PHP 엔진 (mod_php7, libphp7.so)

PHP is an HTML-embedded scripting language.  PHP attempts to make it
easy for developers to write dynamically generated web pages.  PHP
also offers built-in database integration for several commercial
and non-commercial database management systems, so writing a
database-enabled web page with PHP is fairly simple.  The most
common use of PHP coding is probably as a replacement for CGI
scripts.  The mod_php module enables the Apache web server to
understand and process the embedded PHP language in web pages.


### Changes on AnNyung:
1. php 7 업데이트
2. exec_dir (**PHP_INI_SYSTEM**) 기능
 * https://github.com/OOPS-ORG-PHP/mod_execdir/
 * PHP의 shell injection을 **engine level에서 방어**하기 위한 기능
     * 이 기능은 engine level에서 처리를 하기 때문에, php code 쪽에서는 아무런 영향이 없음.
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
 * phpinfo
 * php_uname
 * sys_get_temp_dir
 * phpversion
 * ini_get
 * ini_set
 * ini_get_all
 * get_cfg_var
 * 상단의 적용 function들은 함수명 앞에 prefix로 under bar 3개를 붙이면 호출이 가능함.
 ```php
 <?php
 ___phpinfo();
 ?>
 ```
4. file upload시 image header의 injection code 여부 검사 기능 추가
 * php.ini 에 다음 옵션 추가
     * **upload_image_check** (기본값 On)
         * 업로드 이미지 헤더에 php code가 포함되었는지 검사
         * php code 발견 시에 E_WARNING 발생
     * **upload_image_check_log** (기본값 On) - 업로드 이미지 헤더 검사 관련 로그 기록
     * **upload_image_check_test** (기본값 On)
         * 검사만 하고, 에러 레벨만 리턴(E_WARNING 발생 안함)
         * 검사 결과 감지가 되면 **UPLOAD_ERR_SEC** 반환
     * **upload_image_check_whitelist** - 이미지 헤더 문자열 white list
5. **allow_url_fopen**과 **allow_url_include**를 PHP_INI_ALL로 수정
 * 기본값 OFF이며, php code에서 ini_set으로 변경 가능
6. **allow_include_extension** 옵션 추가 (기본값: **.php**)
 * 등록된 확장자만 php compiler에 의해 compile 됨.
 * include / require 모두 해당
 * 등록된 확장자 파일을 upload 할 경우, **UPLOAD_ERR_ILL** 에러를 반환하고 업로드 되지 않음.
7. **short_open_tag** 기본 값 Off

### Sub packages:
* **php-cli** - php7 cli 인터페이스
* **php-dba** - PHP7 dba 확장
* **php-dblib** - PHP7 dba 확장
* **php-devel** - php7 확장 개발을 위한 파일들
* **php-extension** - php7 shared extension
  * php, php-cli, php-fpm 공용으로 사용
  * /etc/php.d/{apache,cli,fpm}/SHARED.ini 에서 module loading 설정을 해 주어야 함. 기본으로 loading 하지 않음
  ```bash
[root@an3 ~]$ cat /etc/php.d/cli/SHARED.ini
;
; Follow extensions need php-extension package
;
;extension = bcmath.so
;extension = calendar.so
;extension = curl.so
;extension = exif.so
;extension = fileinfo.so
;extension = ftp.so
;extension = gd.so
;extension = gettext.so
;extension = imap.so
;extension = json.so
;extension = ldap.so
;extension = libevent.so
;extension = mcrypt.so
;extension = mysql.so
;extension = mysqli.so
;extension = pdo_mysql.so
;extension = pdo_sqlite.so
;extension = snmop.so
;extension = snmp.so
;extension = soap.so
;extension = sqlite3.so
;extension = wddx.so
;extension = xmlreader.so
;extension = xmlwriter.so
;extension = zip.so
;zend_extension = /usr/lib64/php/extensions/opcache.so
[root@an3 ~]
```
* **php-fpm** - php7 fpm engine
* **php-oci** - PHP7 oci8/pdo_oci 확장
* **php-odbc** - PHP7 odbc, pdo_odbc 확장
* **php-pear** - PHP 확장및 응용 프로그램 저장소 프레임웍
* **php-pgsql** - PHP7 pgsql, pdo_pgsql 확장
* **php-recode** - PHP7 recode 확장
* [**php-common**](pkg-core-php-common.md)
* [**php-fpm-conf**](pkg-core-php-fpm-conf.md)
* [**php-geoip**](pkg-core-php-geoip.md)
* [**php-krisp**](pkg-core-php-krisp.md)
* [**php-nis**](pkg-core-php-nis.md)
* [**php-pecl-apcu**](pkg-addon-php-pecl-apcu.md)
* [**php-pecl-memcache**](pkg-addon-php-pecl-memcache.md)
* [**php-pecl-oauth**](pkg-addon-php-pecl-oauth.md)
* [**php-pecl-xdebug**](pkg-addon-php-pecl-xdebug.md)
* [**php-sqlrelay**](pkg-addon-sqlrelay.md)