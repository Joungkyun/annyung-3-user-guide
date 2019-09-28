# httpd-nis

## Description:

Apache 인증에서 NIS 인증을 사용할 수 있도록 한다.

[httpd](../annyung3-base-packages/pkg-base-httpd.md)의 sub package이다.

## Features:

* NIS에 직접 질의하기 때문에 root권한이 필요 없으며, pam의 영향을 받지 않음
* NIS에 직접 질의하기 때문에 local user의 인증이 안됨.
* Authorization은 mod\_auth\_core에 의존 함.
* 설정 예제

```text
AuthBasicProvider nis

&lt;IfModule mod_auth_nis.c&gt;
    NisDomain NISDOMAINNAME
    NisPwMap  passwd.byname
&lt;/IfModule&gt;

Require valid-user
```

## Reference:

* [http://svn.oops.org/wsvn/Apache.mod\_krisp/trunk/apache2/README](http://svn.oops.org/wsvn/Apache.mod_krisp/trunk/apache2/README)

## Dependencies:

* [httpd](../annyung3-base-packages/pkg-base-httpd.md)

## Sub Packages:

* None

## Related Packages:

* None

