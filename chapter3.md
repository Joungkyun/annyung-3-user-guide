# Chapter 3. HTTP 운영

이 문서는 HTTP를 운영함에 있어 CentOS와 차별/추가/변경된 사항에 대하여 기술 합니다. 특히 httpd(apache) 2.4와 PHP의 경우 큰 변경 사항이 있으니 참고 하시기 바랍니다.

##1. httpd(apache)
### 1. 설정 파일 환경

```bash
[root@an3 httpd]$ tree
.
├── conf
│   ├── httpd.conf
│   └── magic
├── conf.d
│   ├── 00-README
│   ├── LoadModules.conf
│   ├── Security.conf
│   ├── Welcome.conf
│   ├── cgi.conf
│   ├── dav.conf
│   ├── ssl.conf
│   └── userdir.conf
├── logs -> ../../var/log/httpd
├── modules -> ../../usr/lib64/httpd/modules
├── run -> /run/httpd
└── user.d
    ├── Default.conf
    └── vhost.conf

```

안녕의 httpd 설정 파일은 [***httpd-conf***](pkg-core-httpd-conf.md)로 분리 되어 있으며, 위와 같은 구조로 되어 있습니다.

먼저, 종합적인 권고를 우선 말하자면, apache 설정을 추가/변경할 것이 있다면, ***/etc/httpd/user.d/Default.conf*** 에 하십시오! 절대 ***/etc/httpd/conf/httpd.conf***는 수정하지 마십시오!

##2. Nignx
##3. Lighttpd
##4. PHP
##5. Web Access Control List
  1. IP based GEO data
  2. Google Authentificator(Google OTP)를 이용한 2-factor 인증
