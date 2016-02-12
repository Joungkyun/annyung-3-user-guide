# php-fpm-conf

### Description:
[php-fpm](pkg-base-php-fpm.md) 설정 파일 및 init 파일

### Features:
* 설정 파일
 * _/etc/php.d/php-fpm.conf_
 * _/etc/php.d/fpm.d_
 * _/etc/php.d/php-fpm-fcgi.ini_
 * _/etc/php.d/fpm/SHALRED.ini_
* Systemd init 파일
 * _/usr/lib/systemd/system/php-fpm.service_

### Reference:
* https://wiki.php.net/rfc/fpm
* https://wiki.php.net/rfc/fpm/ini_syntax

### Dependencies:
* [php-common](pkg-core-php-common.md)
* [php-fpm](pkg-base-php.md)

### Sub Packages:
* None

### Releated Packages:
* None
