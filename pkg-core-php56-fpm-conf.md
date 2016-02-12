# php56-fpm-conf

### Description:
[php56-fpm](pkg-base-php56-fpm.md) 설정 파일 및 init 파일

### Features:
* 설정 파일
 * _/etc/php56.d/php-fpm.conf_
 * _/etc/php56.d/fpm.d_
 * _/etc/php56.d/php-fpm-fcgi.ini_
 * _/etc/php56.d/fpm/SHALRED.ini_
* Systemd init 파일
 * _/usr/lib/systemd/system/php56-fpm.service_

### Reference:
* https://wiki.php.net/rfc/fpm
* https://wiki.php.net/rfc/fpm/ini_syntax

### Dependencies:
* [php56-common](pkg-core-php56-common.md)
* [php56-fpm](pkg-addon-php56.md)

### Sub Packages:
* None

### Releated Packages:
* None