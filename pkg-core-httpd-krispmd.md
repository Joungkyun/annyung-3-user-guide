# httpd-krisp

### Description:

Remote IP에 대한 국가, ISP 정보를 환경 변수를 생성하며, rewirte rule 등에서 사용할 수 있다.

* Rewirte Rule exmaple:
```httpd
RewriteEngine On
RewriteCond   expr "%{KRISP_COUNTRY_CODE} -strmatch 'KR'"
#RewriteCond  "%{KRISP_COUNTRY_CODE}" "^KR$"
RewriteRule   /.* /kr$0 [L]
```

[httpd](pkg-base-httpd.md)의 sub package이다.

### Reference:

* 모듈 로딩 및 설정: /etc/httpd/conf.d/krisp.conf
* http://svn.oops.org/wsvn/Apache.mod_krisp/trunk/apache2/README