# lighttpd

> 목차
1. 개요
2. lighttpd 설정 파일
3. SSL 설정 및 HTTP2 protocol 지원
4. 안녕에서 제공하는 기능 및 추가 모듈
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
      ssl.pemfile = "/etc/pki/lighttpd/annyung-sample.org.pem"
      ssl.ca-file = "/etc/pki/lighttpd/startssl-sub.class2.server.ca.sha2.pem"
  }

  ```

  [lighttpd](pkg-addon-lighttpd.md)의 인증서는 다음과 같이 생성을 하면 됩니다.
  
  먼저 구동시에 key 암호를 물어보기 때문에, 암호를 제거한 key file을 생성 합니다.
  
  ```bash
  [root@an3 ~]$ openssl rsa -in annyung-sample.org.key -out annyung-sample.org.decrypt.key
  ```
  
  다음 key file과 crt 파일을 합쳐서 [lighttpd](pkg-addon-lighttpd.md)에서 사용할 인증서를 생성 합니다.
  
  ```bash
  [root@an3 ~] cat annyung-sample.org.decrypt.key annyung-sample.org.crt > annyung-sample.org.pem
  ```
  
  chain 인증서가 여러개 일 경우, 인증서 pem 파일(여기서는 annyung-sample.org.pem)에 합치면 됩니다.

  [lighttpd](pkg-addon-lighttpd.md) 1.4 branch는 spdy와 http2를 지원하지 않습니다.


##4. 안녕에서 제공하는 기능 및 추가 모듈

  1. include 지시자에 astrik를 사용할 수 있습니다. 파일이 존재하지 않아도 error가 발생하지 않습니다.
    ```php
    include "conf.d/*.conf"
    ```
    http://redmine.lighttpd.net/issues/1221 참조
    
  2. Apache style의 KeepAlive reponse header를 지원합니다.
    ```
    Connection: Keep-Alive
    Keep-Alive: timeout=60, max=1000
    ```
    http://redmine.lighttpd.net/issues/1284 참조

  3. 404 error page를 URL로 지정할 있습니다. 이는 외부 404 page를 지정할 경우에 200으로 처리되는 문제를 해결하기 위하여 patch 되었습니다.
  4. TCP backlog를 변경할 수 있습니다. (기존은 코드에 1024로 hard coding 되어 있습니다.)
    ```php
    server.backlog = 1024
    ```
  5. mod_dirlisting 의 기능이 향상 되었습니다.
    * dir-listing.gallery  
      * listing시 image 파일이 존재하면 &lt;img&gt; tag로 출력
      * 파일 이름이 cover 또는 preview-000 형식일 경우, cover mode로 출력
    * dir-listing.encoding
      * 문서의 charset을 &lt;meta&gt; tag로 지정
    * dir-listing.html-lang
      * &lt;html lang="VALUE"&gt;
    * dir-listing.urlencode
      * listing file link를 urlencode 하여 출력 (기본값: enable)
    * dir-listing.external-js
      * 외부 javascript url을 삽입
  6. 3rd party module
    * [mod_throttlestatus](http://svn.oops.org/wsvn/Lighttpd.mod_throttlestatus/trunk/throttlestatus.ko.txt) - 디렉토리별 traffic 전송량 표시
    * [mod_url](http://svn.oops.org/wsvn/Lighttpd.mod_url/trunk/README) - URI 문자셋 보정 (apache mod_url의 lighttpd 버전)
    * [mod_net_access](http://svn.oops.org/wsvn/Lighttpd.mod_net_access/trunk/README) - network 접근 제한 고도화
    * [mod_auth_nis](http://svn.oops.org/wsvn/Lighttpd.mod_auth_nis/trunk/README) - NIS 인증 모듈
    * [mod_krisp](http://svn.oops.org/wsvn/Lighttpd.mod_krisp/trunk/README) - IP 관련 Geo data를 환경 변수로 생성. 링크 문서 참조

##5. PHP 연동

  ```php
  server.module         += ("mod_fastcgi")
  fastcgi.debug          = 0
  fastcgi.map-extension  = ( ".kldp" => ".php" )
  fastcgi.server         = (
      ".php" => (
          (
            "host" => "127.0.0.1",
            "port" => 9000,
            "broken-scriptfilename" => "enable",
            "allow-x-send-file" => "enable"
          )
      )
  )
  ```

  lighttpd 문서상, TCP가 아닌 unix domain soecket으로도 fastcgi 연결이 가능하다고 되어 있으나, 실상은 동작하지 않는다. 그러므로 PHP-FPM을 lighttpd와 연동을 하려면 PHP-FPM의 listen을 TCP로 설정해야 한다.

##6. JAVA(tomcat)/Python/Perl 연동

  1. tomcat  
    * proxy module을 이용 (ajp 지원 안함)
    * http://annyung-sample.org/srv/ 를 tomcat 에 연동.

    ```php
      server.module         += ("mod_proxy")
    
      $HTTP["host"] == "annyung-sample.org" {
          $HTTP["url"] =~ "^/srv/" {
              proxy.server = (
                  "" => (
                      "tomcat" => (
                          "host" => "127.0.0.1",
                          "port" => 8080,
                          "fix-redirects" => 1
                      )
                  )
              )
          }
       }
    ```

  2. python django
    * PHP와 동일하게 fastcgi 로 연결합니다. 아래 dango fastcgi 문서에 lighttpd 연동 예제가 포함 되어 있습니다.
    * [django를 fastcgi로 띄우는 구동](https://docs.djangoproject.com/en/1.8/howto/deployment/fastcgi/)
  3. perl
    1. fastcgi 연동
       * http://nginxlibrary.com/perl-fastcgi/ 문서에서 perl fastcgi wrapper 부분을 참조 하십시오.
       * lighttpd 설정은 PHP나 Python django와 동일하게 하면 됩니다.
    2. CGI 연동 - ***4. CGI 연동***을 참조 하십시오.
  4. CGI 연동
    ```php
    server.module      += ("mod_cgi")
    server.breakagelog  = "/var/log/lighttpd/breakage.log"
    cgi.assign = (
        ".pl"  => "/usr/bin/perl",
        ".py"  => "/usr/bin/python",
        ".cgi" => "/usr/bin/perl"
    )
    ```
    http://redmine.lighttpd.net/projects/1/wiki/docs_modcgi 참조

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
