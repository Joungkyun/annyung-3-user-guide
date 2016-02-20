# lighttpd

> 목차
1. 개요
2. lighttpd 설정 파일
3. SSL 설정 및 HTTP2 protocol 지원
4. 안녕에서 제공하는 추가 모듈
5. PHP 연동
6. JAVA(tomcat)/Python/Perl 연동
7. lighttpd 구동

##1. 개요

  안녕 리눅스의 [lighttpd](pkg-addon-lighttpd.md)는 1.4 branch를 제공 합니다.
  
  [lighttpd](pkg-addon-lighttpd.md)를 사용하는 것 보다 [nginx](chapter3-2-nginx.md)를 사용하는 것을 권고 합니다.
  
  [lighttpd](pkg-addon-lighttpd.md)는 1.4 branch는 현재 7여년 동안 개발이 정체되어 있고 새로운 기술들이 반영되어진 2.0이 아직 출시 단계에도 이르지 못한 상황이므로, 동일하게 single thread model인 [nginx](chapter3-2-nginx.md)를 선택하는 것이 더 좋습니다.

##2. lighttpd 설정 파일

  ```bash
  [root@an3 ~]$ tree /etc
  /etc
  ├── lighttpd
  │   ├── autoindex.conf
  │   ├── conf.d
  │   │   ├── Default.conf
  │   │   ├── README
  │   │   └── vhost.conf
  │   ├── lighttpd.conf
  │   └── mime.conf
  ├── logrotate.d
  │   └── lighttpd
  └── sysconfig
      ├── lighttpd
      └── lighttpd-monitor
  ```

  * ***/etc/lighttpd/lighttpd.conf*** 는 동작에 필요한 최소한의 설정만을 가지고 있습니다. <u>수정하지 않도록 합니다.</u>.
  * ***/etc/lighttpd/conf.d***에 사용자 설정을 추가/변경 하도록 합니다. ***lighttpd.conf***의 설정을 변경하고자 한다면, 이곳에서 설정을 하면 overwrite가 됩니다.
  * ***/etc/logroate.d/lighttpd***파일에 log rotate 설정이 있습니다.
  * ***/etc/sysconfig/lighttpd*** 파일에 lighttpd 구동을 위한 설정이 있습니다.
  * ***/etc/sysconfig/lighttpd-monitor*** 는 ***/usr/sbin/lighttpd-monitor*** 명령을 실행하는데 필요한 옵션값들이 설정 되어 있습니다. 이에 관련해서는 [웹서버 모니터링](chapter3-6-web-monitor.md) 문서에서 기술 합니다.


##3. SSL 설정 및 HTTP2 protocol 지원

  설정 파일의 예제는 다음과 같습니다.
  
  ```php
  $SERVER["socket"] == ":443" {
      ssl.engine             = "enable"
      ssl.use-sslv2          = "disable"
      ssl.use-sslv3          = "disable"
      ssl.honor-cipher-order = "enable"
      ssl.cipher-list = "ECDH+AESGCM:ECDH+AES256:ECDH+AES128:ECDH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL"
      ssl.pemfile = "/etc/pki/lighttpd/oops.org.pem"
      ssl.ca-file = "/etc/pki/lighttpd/startssl-sub.class2.server.ca.sha2.pem"
  }

  ```

  [lighttpd](pkg-addon-lighttpd.md)의 인증서는 다음과 같이 생성을 하면 됩니다.
  
  먼저 구동시에 key 암호를 물어보기 때문에, 암호를 제거한 key file을 생성 합니다.
  
  ```bash
  [root@an3 ~]$ openssl rsa -in oops.org.key -out oops.key.decrypt.key
  ```
  
  다음 key file과 crt 파일을 합쳐서 [lighttpd](pkg-addon-lighttpd.md)에서 사용할 인증서를 생성 합니다.
  
  ```bash
  [root@an3 ~] cat oops.org.decrypt.key oops.org.crt > oops.org.pem
  ```

  [lighttpd](pkg-addon-lighttpd.md) 1.4 branch는 spdy와 http2를 지원하지 않습니다.


##4. 안녕에서 제공하는 추가 모듈



##5. PHP 연동



##6. JAVA(tomcat)/Python/Perl 연동

  안녕 리눅스에서 특별히 반영한 것이 없습니다. 기존에 하시던 방법이 있으면 그대로 하시면 무방 합니다. 또한, 인터넷 검색을 활용 하십시오. 

##7. lighttpd 구동

  간단한 apcahe control 방법에 대하여 기술 합니다.

  * 부팅시 lighttpd 시작하도록 설정
  ```bash
  [root@an3 ~]$ service lighttpd enable
  ```
  * 부팅시 lighttpd 시작 하지 않도록 설정
  ```bash
  [root@an3 ~]$ service lighttpd disable
  ```
  * lighttpd 시작
  ```bash
  [root@an3 ~]$ service lighttpd start
  ```
  * lighttpd 정지
  ```bash
  [root@an3 ~]$ service lighttpd stop
  ```
  * lighttpd 재시작
  ```bash
  [root@an3 ~]$ service lighttpd restart
  ```
  * lighttpd 상태 보기
  ```bash
  [root@an3 ~]$ service lighttpd status
  ```
