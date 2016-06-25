# httpd-url

### Description:

Client 요청 URI와 서버 파일간의 character set 문제가 있을 경우 보정을 한다.

[httpd](pkg-base-httpd.md)의 sub package이다.

### Features:

* client에서 UTF8 문자셋으로 URI를 요청했을 경우, 서버의 파일이 EUC-KR이라면 mod_url은 내부적으로 원 요청으로 확인을 한 후, 없으면 EUC-KR로 변환을 하여 다시 체크를 함.
* 설정 예제

```httpd
&lt;IfModule redurl_module&gt;
    CheckURL On
    ServerEncoding EUC-KR
    ClientEncoding UTF-8
&lt;/IfModule&gt;
```

### Reference:

* https://github.com/Joungkyun/mod_url/blob/master/apache2/README

### Dependencies:
* [httpd](pkg-base-httpd.md)

### Sub Packages:
* None

### Related Packages:
* None