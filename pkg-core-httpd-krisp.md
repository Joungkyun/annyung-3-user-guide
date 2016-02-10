# httpd-krisp

### Description:

Remote IP에 대한 국가, ISP 정보를 환경 변수를 생성하며, rewirte rule 등에서 사용할 수 있다.

### Features:

* 모듈 로딩 및 설정: /etc/httpd/conf.d/krisp.conf
* 생성된 환경 변수는 httpd.conf 및 PHP등의 언어에서 web server 환경 변수로 호출이 가능 함.
* Rewirte Rule exmaple:
```httpd
RewriteEngine On
RewriteCond   expr "%{KRISP_COUNTRY_CODE} -strmatch 'KR'"
#RewriteCond  "%{KRISP_COUNTRY_CODE}" "^KR$"
RewriteRule   /.* /kr$0 [L]
```

### Reference:

* http://svn.oops.org/wsvn/Apache.mod_krisp/trunk/apache2/README

### Dependencies:

* [libkrisp](pkg-core-libkrisp.md)
* [httpd](pkg-base-httpd.md)

### Sub Packages:
* None

### Releated Packages:
* [httpd-krisp](pkg-core-httpd-krisp.md)
* [perl-krisp](pkg-core-perl-krisp.md)
* [php-krisp](pkg-core-php-krisp.md)
* [php-pecl-krisp](pkg-core-php-pecl-krisp.md)
* [python-krisp](pkg-core-python-krisp.md)