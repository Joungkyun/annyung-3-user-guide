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

### Reference:
* None

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