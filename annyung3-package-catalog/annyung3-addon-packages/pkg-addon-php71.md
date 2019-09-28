# php71

## Description:

PHP is an HTML-embedded scripting language. PHP attempts to make it easy for developers to write dynamically generated web pages. PHP also offers built-in database integration for several commercial and non-commercial database management systems, so writing a database-enabled web page with PHP is fairly simple. The most common use of PHP coding is probably as a replacement for CGI scripts. The mod\_php module enables the Apache web server to understand and process the embedded PHP language in web pages.

## Features:

1. PHP 7.1 compatible 패키지
2. Apache module\(**libphp7.so**\)은 지원하지 않고, cli와 FPM만 지원합니다. Fastcgi protocol을 이용하여 **php71-fpm**과 연결하여야 합니다.
3. php71-cli 설정 파일
   * _/etc/php71.d/php-cli.ini_
   * _/etc/php71.d/cli/SHARED.ini_
   * php-cli.ini는 수정을 하지 말고, 변경할 옵션을 _/etc/php71.d/cli/SHARED.ini_ 에 하는 것을 권장함
4. php71-fpm 설정 파일
   * FPM 구동 설정 파일
     * _/etc/php71.d/fpm.conf_
     * _/etc/php71.d/fpm.d_
   * FPM 설정 파일
     * _/etc/php71.d/php-fpm-fcgi.ini_
     * _/etc/php71.d/fpm/SHARED.ini_
     * php-fpm-fcgi.ini는 수정을 하지 말고, 변경할 옵션을 _/etc/php71.d/fpm/SHARED.ini_ 에 하는 것을 권장함
5. 성능
   * PHP VM type을 _**GOTO**_ mode로 빌드 \(성능 20% 향상\)
   * _**realpath\_cache\_force**_ ini 옵션 제공
     * _**openbase\_dir**_ 사용시에, realpath\_cache 기능을 강제로 사용하게 하여 30% 정도의 성능을 향상
   * [https://my.oops.org/173](https://my.oops.org/173) 참조

## Reference:

1. exec\_dir \(**PHP\_INI\_SYSTEM**\) 기능
   * PHP의 shell injection을 **engine level에서 방어**하기 위한 기능
     * 이 기능은 engine level에서 처리를 하기 때문에, php code 쪽에서는 아무런 영향이 없음.
   * 2005년 부터 KLDP와 N사 T사의 core system에 적용되어 검증
   * PHP 5.4 이전의 safe\_mode\_exec\_dir을 safe\_mode가 아닌 경우에도 사용할 수 있도록 수정하고 command 치환 parser를 확장
   * 기본값 _/var/lib/php71/bin_
     * 안녕의 PHP 에서 system 함수를 사용하려면 사용하려면 command가 /var/lib/php56/bin 에 soft link나 복사 되어야 함.
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

   * 참조: 
     * [http://kldp.org/node/45576](http://kldp.org/node/45576)
     * [https://github.com/OOPS-ORG-PHP/mod\_execdir](https://github.com/OOPS-ORG-PHP/mod_execdir)
   * 적용 functions
     * 내부적으로 php\_exec API를 호출하는 function들
     * [system](http://php.net/manual/kr/function.system.php)
     * [exec](http://php.net/manual/kr/function.exec.php)
     * [passthru](http://php.net/manual/kr/function.passthru.php)
     * [popen](http://php.net/manual/kr/function.popen.php)
     * [escapeshellcmd](http://php.net/manual/kr/function.escapeshellcmd.php)
     * [pcntl\_exec](http://php.net/manual/kr/function.pcntl-exec.php)
     * [backtick operator](http://php.net/manual/kr/language.operators.execution.php)
2. disable\_functions 기본 적용
   * phpinfo
   * php\_uname
   * sys\_get\_temp\_dir
   * phpversion
   * ini\_get
   * ini\_set
   * ini\_get\_all
   * get\_cfg\_var
   * 상단의 적용 function들은 함수명 앞에 prefix로 under bar 3개를 붙이면 호출이 가능함.

     ```php
     <?php
     ___phpinfo();
     ?>
     ```
3. file upload시 image header의 injection code 여부 검사 기능 추가
   * php.ini 에 다음 옵션 추가
     * **upload\_image\_check** \(기본값 On\)
       * 업로드 이미지 헤더에 php code가 포함되었는지 검사
       * php code 발견 시에 E\_WARNING 발생
     * **upload\_image\_check\_log** \(기본값 On\) - 업로드 이미지 헤더 검사 관련 로그 기록
     * **upload\_image\_check\_test** \(기본값 On\)
       * 검사만 하고, 에러 레벨만 리턴\(E\_WARNING 발생 안함\)
       * 검사 결과 감지가 되면 **UPLOAD\_ERR\_SEC** 반환
     * **upload\_image\_check\_whitelist** - 이미지 헤더 문자열 white list
4. **allow\_url\_fopen**과 **allow\_url\_include**를 PHP\_INI\_ALL로 수정
   * 기본값 OFF이며, php code에서 ini\_set으로 변경 가능
5. **allow\_include\_extension** 옵션 추가 \(기본값: **.php**\)
   * 등록된 확장자만 php compiler에 의해 compile 됨.
   * include / require 모두 해당
   * 등록된 확장자 파일을 upload 할 경우, **UPLOAD\_ERR\_ILL** 에러를 반환하고 업로드 되지 않음.
6. **short\_open\_tag** 기본 값 Off
   * short\_open\_tag 가 Off 이더라도, _**&lt;?=$var?&gt;**_ 구문은 사용 가능.

## Dependencies:

* None

## Sub Packages:

* **php71-cli** - php 5.6 cli 인터페이스
* **php71-dba** - PHP 5.6 dba 확장
* **php71-dblib** - PHP 5.6 dba 확장
* **php71-devel** - php 5.6 확장 개발을 위한 파일들
* **php71-extension** - php 5.6 shared extension
  * php71-cli, php71-fpm 공용으로 사용
  * /etc/php71.d/{cli,fpm}/SHARED.ini 에서 module loading 설정을 해 주어야 함. 기본으로 loading 하지 않음

    ```bash
    [root@an3 ~]$ cat /etc/php71.d/cli/SHARED.ini
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
    ;zend_extension = /usr/lib64/php71/extensions/opcache.so
    [root@an3 ~]
    ```
* **php71-fpm** - php 7.1 fpm engine
* **php71-oci** - PHP 7.1 oci8/pdo\_oci 확장
* **php71-odbc** - PHP 7.1 odbc, pdo\_odbc 확장
* **php-pear** - PHP 7.1 확장및 응용 프로그램 저장소 프레임웍
* **php71-pgsql** - PHP 7.1 pgsql, pdo\_pgsql 확장
* **php71-recode** - PHP 7.1 recode 확장
* [**php71-common**](../annyung3-core-packages/pkg-core-php71-common.md)
* [**php71-fpm-conf**](../annyung3-core-packages/pkg-core-php71-fpm-conf.md)
* [**php71-geoip**](../annyung3-core-packages/pkg-core-php71-geoip.md)
* [**php71-krisp**](../annyung3-core-packages/pkg-core-php71-krisp.md)
* [**php71-nis**](../annyung3-core-packages/pkg-core-php71-nis.md)
* [**php71-pecl-apcu**](https://github.com/joungkyun/annyung-3-user-guide/tree/dae3c13e1446e9d689ecf1babc8ac28b5c437457/pkg-addon-php71-pecl-apcu.md)
* [**php71-pecl-oauth**](pkg-addon-php71-pecl-oauth.md)
* [**php71-pecl-xdebug**](pkg-addon-php71-pecl-xdebug.md)
* [**php71-sqlrelay**](https://github.com/joungkyun/annyung-3-user-guide/tree/dae3c13e1446e9d689ecf1babc8ac28b5c437457/pkg-addon-php71-sqlrelay.md)

## Related Packages:

* [php](../annyung3-base-packages/pkg-base-php.md)

