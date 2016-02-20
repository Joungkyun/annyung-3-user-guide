# Chapter 3. HTTP 운영

이 문서는 HTTP를 운영함에 있어 CentOS와 차별/추가/변경된 사항에 대하여 기술 합니다. 특히 httpd(apache) 2.4와 PHP의 경우 큰 변경 사항이 있으니 참고 하시기 바랍니다.

##1. httpd(apache)
### 1. 주의 사항

1. 안녕 리눅스의 httpd의 기본 MPM은 event 입니다.
  1. mod_php7 을 사용하기 위해서는 MPM을 prefork로 변경해야 합니다.
  2. mod_php7 보다는 php-fpm 구성을 권고 합니다. 자세한 사항은 PHP 섹션을 참고 하십시오.
  3. CGI 설정시, MPM이 event나 worker일 경우, 
2. user_dir이 기본 off 로 설정 되어 있습니다. (기본으로 /~user 접근이 차단 되어 있습니다.)
3. ***/etc/httpd/conf.d/Security.conf*** 를 꼭 확인 하십시오.  
  웹 공격에 취약한 접근에 대하여 미리 접근을 차단하고 있습니다. <u>이 설정은 서비스 운영 시에 꼭 확인</u>을 하시기 바랍니다. 이 설정 때문에 원하는 동작이 되지 않을 수 있습니다. 


### 2. 설정 파일 환경

> 먼저, 종합적인 권고를 우선 말하자면, apache 설정을 추가/변경할 것이 있다면, ***/etc/httpd/user.d/Default.conf*** 에 하십시오! 절대 ***/etc/httpd/conf/httpd.conf***는 수정하지 마십시오! 모듈 설정은 ***/etc/httpd.conf.d***에서 운영에 관한 설정은 ***/etc/httpd/user.d*** 에서 하십시오.

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
│   └── userdir.conf
├── logs -> ../../var/log/httpd
├── modules -> ../../usr/lib64/httpd/modules
├── run -> /run/httpd
└── user.d
    ├── Default.conf
    └── vhost.conf

```

안녕의 httpd 설정 파일은 [***httpd-conf***](pkg-core-httpd-conf.md)로 분리 되어 있으며, 위와 같은 구조로 되어 있습니다.

 * ***/etc/httpd/conf***
   * 전통적인 httpd의 설정 경로 입니다.
   * ***httpd.conf***는 apache를 시작하면 <u>동작이 가능한 최소한의 설정으로 구성</u>이 되어 있습니다.
   * ***httpd.conf***에 설정 되어 있는 지시자들은 모두 설정값이 ***overwrite***가 가능한 지시자들 입니다.
   * <u>***httpd.conf***를 절대 수정 하지 마십시오!</u>
 * ***/etc/httpd/conf.d***
   * apache module 들에 대한 설정을 가지고 있습니다.
   * ***LoadModules.conf***
     * httpd 기본 패키지에 포함된 module load 설정 이며, 별 다른 기본 설정이 필요 없는 모듈들이 등록 되어 있습니다.
     * 최소한의 동작만을 위한 module들이 load 되어 있기 때문에, 필요한 모듈들을 주석을 해제하여 load 시켜 주어야 할 수 있습니다.
 * ***/etc/httpd/user.d***
   * 모듈 설정과 관련 없는 사이트 설정들을 포함합니다. 
 * ***/etc/httpd/httpd.conf*** 에서 ***/etc/httpd/conf.d/*.conf***를 include 한 다음 ***/etc/httpd/user.d/*.conf***를 include 하기 때문에 중복되는 지시자 중 ***/etc/httpd/user.d/***에 설정된 지시자가 최종 반영이 됩니다.






##2. Nignx
##3. Lighttpd
##4. PHP
##5. Web Access Control List
  1. IP based GEO data
  2. Google Authentificator(Google OTP)를 이용한 2-factor 인증
