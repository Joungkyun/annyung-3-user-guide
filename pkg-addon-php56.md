# php56

### Description:
PHP is an HTML-embedded scripting language.  PHP attempts to make it
easy for developers to write dynamically generated web pages.  PHP
also offers built-in database integration for several commercial
and non-commercial database management systems, so writing a
database-enabled web page with PHP is fairly simple.  The most
common use of PHP coding is probably as a replacement for CGI
scripts.  The mod_php module enables the Apache web server to
understand and process the embedded PHP language in web pages.

### Features:
1. PHP 5.6 compatible 패키지
2. Apache module(**libphp5.so**)은 지원하지 않고, cli와 FPM만 지원합니다. Fastcgi protocol을 이용하여 **php56-fpm**과 연결하여야 합니다.
3. php56-cli 설정 파일
 * _/etc/php56.d/php-cli.ini_
 * _/etc/php56.d/cli/SHARED.ini_
 * <u>php-cli.ini는 수정을 하지 말고</u>, 변경할 옵션을 _/etc/php56.d/cli/SHARED.ini_ 에 하는 것을 권장함
4. php56-fpm 설정 파일
 * FPM 구동 설정 파일
    * _/etc/php56.d/fpm.conf_
    * _/etc/php56.d/fpm.d_
 * FPM 설정 파일
    * _/etc/php56.d/php-fpm-fcgi.ini_
    * _/etc/php56.d/fpm/SHARED.ini_
    * <u>php-fpm-fcgi.ini는 수정을 하지 말고</u>, 변경할 옵션을 _/etc/php56.d/fpm/SHARED.ini_ 에 하는 것을 권장함

### Reference:
1. PHP 5.3 호환 기능
 * PHP 5.4에서 deprecated 되었거나 remove된 기능들을 지원
 * _/etc/php56.d/{cli,fpm}/php53compatible.ini_

  ```ini
  ; PHP 5.3 호환 모드 지원
  ;
  ; 이 파일의 설정을 주석처리할 경우, php.ini 또는 php-cli.ini 또는
  ; php-fcig.ini 의 기본값으로 동작을 하게 되니 주석 처리시에는 주의해야
  ; 합니다.
  ;
  ; On으로 설정시 php 5.4에서 제거되었거나 변경된 기능을 5.3과 같이 동작함
  ;
  ; . allow_call_time_pass_reference 지시자 사용 가능 (Default: Off)
  ; . magic_quotes_gpc, magic_quotes_runtime, magic_quotes_sybase 지시자
  ;   및 magic_quotes 관련 함수 사용 가능 (기본값: Off)
  ; . NULL, false, 빈문자열의 값을 가진 변수에 object property를 추가할
  ;   경우에도 E_WARNING 에러 메시지 발생 하지 않음
  ; . TZ 환경 변수로 timezone 지정 가능
  ; . array_combine() 함수에서 key array가 비었을 경우 false 반환
  ; . 5.4에서 제거된 다음의 함수 사용 가능 (E_DEPRECATED level 에러 처리)
  ;   session_is_registered(), session_register(), session_unregister()
  ;   mysqli_bind_param(), mysqli_bind_result(), mysqli_client_encoding(),
  ;   mysqli_fetch(), mysqli_param_count(), mysqli_get_metadata(),
  ;   mysqli_send_long_data(), mysqli::client_encoding()
  ;
  ; Default Value: Off
  ; Development Value: Off
  ; Production Value: Off
  ;
  php53_compatible = Off

  ; This directive allows you to enable and disable warnings which PHP will issue
  ; if you pass a value by reference at function call time. Passing values by
  ; reference at function call time is a deprecated feature which will be removed
  ; from PHP at some point in the near future. The acceptable method for passing a
  ; value by reference to a function is by declaring the reference in the functions
  ; definition, not at call time. This directive does not disable this feature, it
  ; only determines whether PHP will warn you about it or not. These warnings
  ; should enabled in development environments only.
  ; Default Value: On (Suppress warnings)
  ; Development Value: Off (Issue warnings)
  ; Production Value: Off (Issue warnings)
  ; http://php.net/allow-call-time-pass-reference
  ;
  ; This directive has dependency with 'php53_compatible=On'
  allow_call_time_pass_reference = Off

  ; Magic quotes are a preprocessing feature of PHP where PHP will attempt to
  ; escape any character sequences in GET, POST, COOKIE and ENV data which might
  ; otherwise corrupt data being placed in resources such as databases before
  ; making that data available to you. Because of character encoding issues and
  ; non-standard SQL implementations across many databases, it's not currently
  ; possible for this feature to be 100% accurate. PHP's default behavior is to
  ; enable the feature. We strongly recommend you use the escaping mechanisms
  ; designed specifically for the database your using instead of relying on this
  ; feature. Also note, this feature has been deprecated as of PHP 5.3.0
  ; Default Value: On
  ; Development Value: Off
  ; Production Value: Off
  ; http://php.net/magic-quotes-gpc
  ;
  ; This directive has dependency with 'php53_compatible=On'
  magic_quotes_gpc = Off

  ; Magic quotes for runtime-generated data, e.g. data from SQL, from exec(), etc.
  ; http://php.net/magic-quotes-runtime
  ;
  ; This directive has dependency with 'php53_compatible=On'
  magic_quotes_runtime = Off

  ; Use Sybase-style magic quotes (escape ' with '' instead of \').
  ; http://php.net/magic-quotes-sybase
  ;
  ; This directive has dependency with 'php53_compatible=On'
  magic_quotes_sybase = Off
  ```
2. exec_dir (**PHP_INI_SYSTEM**) 기능
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

### Dependencies:
* None

### Sub Packages:
* **php56-cli** - php 5.6 cli 인터페이스
* **php56-dba** - PHP 5.6 dba 확장
* **php56-dblib** - PHP 5.6 dba 확장
* **php56-devel** - php 5.6 확장 개발을 위한 파일들
* **php56-extension** - php 5.6 shared extension
  * php56-cli, php56-fpm 공용으로 사용
  * /etc/php56.d/{cli,fpm}/SHARED.ini 에서 module loading 설정을 해 주어야 함. 기본으로 loading 하지 않음
  ```bash
[root@an3 ~]$ cat /etc/php.d/cli/SHARED.ini
;
; Follow extensions need php-extension package
;
;extension = bcmath.so
;extension = calendar.so
;extension = curl.so
;extension = event.so
;extension = exif.so
;extension = fileinfo.so
;extension = ftp.so
;extension = gd.so
;extension = gettext.so
;extension = gmp.so
;extension = imap.so
;extension = json.so
;extension = ldap.so
;extension = libevent.so
;extension = mcrypt.so
;extension = memcache.so
;extension = mysql.so
;extension = mysqli.so
;extension = pdo_mysql.so
;extension = pdo_sqlite.so
;extension = snmp.so
;extension = soap.so
;extension = sqlite.so
;extension = sqlite3.so
;extension = wddx.so
;extension = xmlreader.so
;extension = xmlwriter.so
;extension = zip.so
;zend_extension = /usr/lib64/php/extensions/opcache.so
[root@an3 ~]
```
* **php56-fpm** - php 5.6 fpm engine
* **php56-oci** - PHP 5.6 oci8/pdo_oci 확장
* **php56-odbc** - PHP 5.6 odbc, pdo_odbc 확장
* **php-pear** - PHP 5.6 확장및 응용 프로그램 저장소 프레임웍
* **php56-pgsql** - PHP 5.6 pgsql, pdo_pgsql 확장
* **php56-recode** - PHP 5.6 recode 확장
* [**php56-common**](pkg-core-php56-common.md)
* [**php56-fpm-conf**](pkg-core-php56-fpm-conf.md)
* [**php56-geoip**](pkg-core-php56-geoip.md)
* [**php56-krisp**](pkg-core-php56-krisp.md)
* [**php56-nis**](pkg-core-php56-nis.md)
* [**php56-pecl-apcu**](pkg-addon-php56-pecl-apcu.md)
* [**php56-pecl-oauth**](pkg-addon-php56-pecl-oauth.md)
* [**php56-pecl-xdebug**](pkg-addon-php56-pecl-xdebug.md)
* [**php56-sqlrelay**](pkg-addon-sqlrelay.md)

### Releated Packages:
* None