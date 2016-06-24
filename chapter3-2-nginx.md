#nginx 기본 운영

> 목차
1. 개요
2. nginx 설정 파일
3. SSL 설정 및 HTTP2 protocol 지원
4. 안녕에서 제공하는 추가 모듈
5. PHP 연동
6. JAVA(tomcat)/Python/Perl 연동
7. nginx 구동

##1. 개요

  안녕 리눅스에서 제공하는 nginx는 1.10 ***stable*** branch를 제공합니다.
  
  참고로, 안녕 리눅스에서 제공하는 error page는 <u>공식적인 서비스에 활용하기에 적합하지 않은 표현을 사용</u>하고 있습니다. 공식적인 서비스 사용 시에는 꼭 에러 페이지들을 별도로 만드는 것을 권고합니다.

##2. ningx 설정 파일

```bash
[root@an3 ~]$ tree /etc
etc
├── logrotate.d
│   └── nginx
├── nginx
│   ├── common.d
│   │   ├── core-alias.conf
│   │   ├── core-logging.conf
│   │   └── core-secure.conf
│   ├── conf.d
│   │   ├── Default.conf
│   │   ├── ssl.conf
│   │   └── vhost.conf
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── nginx.conf
│   ├── modules.conf
│   ├── params
│   │   ├── fastcgi.conf
│   │   ├── fastcgi_params
│   │   ├── scgi_params
│   │   └── uwsgi_params
│   └── win-utf
└── sysconfig
    └── nginx
```

1. ***/etc/ngninx/nginx.conf***는 구동을 위한 최소한의 설정만을 가지고 있습니다. 그러므로 <u>이 파일을 수정하지 마십시오</u>.
2. 사용자 설정은 ***/etc/nginx/conf.d*** 에서 하십시오.
3. ***/etc/logrotate.d/nginx*** 에서 log ratation 설정을 하고 있습니다. 기본으로 10일치의 로그를 남기도록 되어 있습니다.
4. ***/etc/sysconfig/nginx***에서 init script에 필요한 옵션을 설정하고 있습니다.
5. ***server*** block의 마지막에 아래 예제와 같이 ___common.d/*.conf___를 include 하십시오.
```nginx
server {
    listen       443 ssl spdy;
    server_name  annyung-sample.org;
    root         /home/httpd/annyung-smaple.org;
    index        index.html;

    # ** 중략 **

    # Load configuration files for the default server block.
    include /etc/nginx/common.d/*.conf;
}
```
6. ***/etc/nginx/common.d/core-secure.conf***에서 취약한 웹 접근을 막는 설정이 있습니다. 운영에 문제가 될 수 있는 설정이니, 서비스 전에 꼭 확인 하십시오.

##3. SSL 설정 및 HTTP2 protocol 지원

```nginx
server {
    listen       443 ssl http2;
    server_name  annyung-sample.org;
    root         /home/httpd/annyung-smaple.org;
    index        index.html;

    # for ssl certificate configuration
    ssl                 on
    ssl_certificate     annyung-sample.org.pem
    ssl_certificate_key annyung-sample.org.decrypt.key
    
    ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers          ECDH+AESGCM:ECDH+AES256:ECDH+AES128:ECDH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL
    ssl_prefer_server_ciphers on;

    # Load configuration files for the default server block.
    include /etc/nginx/common.d/*.conf;
}
```

  위의 설정을 참고 하십시오. 이 설정으로 [SSLlabs](https://www.ssllabs.com/)의 ***A-*** 등급을 받을 수 있습니다.
  
  nginx의 인증서는 chain 인증서가 존재할 경우, 인증서와 chain 인증서를 합쳐서 만들어야 합니다.
  
  다음은 startssl class 2 chain 인증서를 이용하여 만드는 경우 입니다.
  
  ```bash
  [root@an3 ~]$ cat annyung-sample.org.crt startssl-sub.class2.server.ca.sha2.pem > annyung-sample.org.pem
  ```
  
  key 파일의 암호를 제거하지 않으면, ngninx 구동 시에 key 암호를 입력해야 합니다. 이를 해결하기 위해서 암호를 제거한 key 파일을 설정 합니다.
  
  ```bash
  [root@an3 ~]$ openssl rsa -in annyung-sample.org.key -out annyung-sample.org.decrypt.key
  ```
  
  
  또한, ningx 는 http2 protocol을 1.9 main line에서 지원하고 있습니다. 안녕 리눅스는 ***stable*** 버전인 1.10을 제공하고 있으므로 h2c protocol 을 지원합니다. 이전에 1.8을 설치했었다면, ***spdy*** 옵션을 ***http2***로 변경 하십시오.
  
  안녕 리눅스의 ssl 설정 예제는 기본으로 h2c(http2)를 사용 하도록 되어 있습니다.

##4. 안녕에서 제공하는 추가 모듈

 1. [URL module](https://github.com/vozlt/nginx-module-url)  
    apache mod_url 과 동일한 동작을 합니다.
 2. [krisp module](https://github.com/vozlt/nginx-module-krisp)  
    nginx libkrisp module
 3. [vts module](https://github.com/vozlt/nginx-module-vts)  
    가상 호스트별 tracffic 상태를 확인 할 수 있습니다.
 4. [fancyindex](https://www.nginx.com/resources/wiki/modules/fancy_index/)  
    directory listing을 이쁘게 해 줍니다.
    안녕 리눅스에는 다음의 지시자가 추가 되었습니다.
    * fancyindex_readme  
      지정된 파일을 listing 아래에 출력 합니다. Github의 README.md 와 같이 보여진다고 생각하면 됩니다. 마단 markup은 지원하지 않고 HTML tag를 지원합니다.
    * fancyindex_ignore_user  
      지정된 user권한을 가진 파일/디렉토리는 listing 하지 않습니다.
    * fancyindex_ignore_group  
      지정된 group 권한을 가진 파일/디렉토리는 listing 하지 않습니다.
 5. [Headers more](https://www.nginx.com/resources/wiki/modules/headers_more/)
    Response header에 header를 추가 하거나 제거 합니다.

##5. PHP 연동

  Nginx에서의 PHP연동은 fastcgi protocol을 이용하여 PHP-FPM과 연동을 합니다. PHP-FPM 구동은 안녕 리눅스 사용자 가이드 [3.1.4 PHP](chapter3-4-php.md) 문서를 참조 하십시오.
  
  nginx의 fastcgi 연동은 case by case의 경우가 많습니다. 여기서 모든 설명을 하기 힘드니, 여기서는 http://wiki.kldp.org (moniwiki) 의 실제 설정 예제를 참조 합니다.
  
  ```nginx
  server {
    listen       443 ssl spdy;
    server_name  wiki.kldp.org;
    root         /home/wiki;
    index        index.html;

    # for ssl
    include      /etc/nginx/cert.d/kldp.org.conf;

    location ~ ^/(data|conf(ig)?|bin|inc(lude)?|plugins?|wikiseed)/ {
        deny     all;
    }

    # Load configuration files for the default server block.
    include /etc/nginx/common.d/*.conf;

    location ~ \.php($|/) {
        include  params/fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm.sock;

        if ( !-f $document_root$fastcgi_script_name ) {
            return 404;
        }

        fastcgi_index index.php;
        fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_read_timeout 60;
    }
  }

  ```

##6. JAVA(tomcat)/Python/Perl 연동

  안녕 리눅스에서 특별히 반영한 것이 없습니다. 기존에 하시던 방법이 있으면 그대로 하시면 무방 합니다. 또한, 인터넷 검색을 활용 하십시오. 

##7. nginx 구동

  간단한 nginx control 방법에 대하여 기술 합니다.

  * 부팅시 nginx 시작하도록 설정
  ```bash
  [root@an3 ~]$ service nginx enable
  [root@an3 ~]$ # 또는
  [root@an3 ~]$ ntsysv-systemd
  ```
  * 부팅시 nginx 시작 하지 않도록 설정
  ```bash
  [root@an3 ~]$ service nginx disable
  [root@an3 ~]$ # 또는
  [root@an3 ~]$ ntsysv-systemd
  ```
  * nginx 시작
  ```bash
  [root@an3 ~]$ service nginx start
  ```
  * nginx 정지
  ```bash
  [root@an3 ~]$ service nginx stop
  ```
  * nginx 재시작
  ```bash
  [root@an3 ~]$ service nginx restart
  ```
  * nginx 상태 보기
  ```bash
  [root@an3 ~]$ service nginx status
  ```
