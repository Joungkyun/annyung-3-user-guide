# httpd-nis

### Description:

Apache 인증에서 NIS 인증을 사용할 수 있도록 한다.

[httpd](pkg-base-httpd.md)의 sub package이다.

### Features:

* NIS에 직접 질의하기 때문에 root권한이 필요 없으며, pam의 영향을 받지 ㅇ낳음
* NIS에 직접 질의하기 때문에 local user의 인증이 안됨.
* Authorization은 mod_auth_core에 의존 함.
* 설정 예제

```httpd
AuthBasicProvider nis

&lt;IfModule mod_auth_nis.c&gt;
    NisDomain NISDOMAINNAME
    NisPwMap  passwd.byname
&lt;/IfModule&gt;

Require valid-user
```

### Reference:

* http://svn.oops.org/wsvn/Apache.mod_krisp/trunk/apache2/README




### Releated Packages:
* [httpd](pkg-base-httpd.md)의 sub package
