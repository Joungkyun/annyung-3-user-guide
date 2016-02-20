# nginx

##1. 개요

  안녕 리눅스에서 제공하는 nginx는 1.8 ***stable*** branch를 제공합니다.
  
  참고록, 안녕 리눅스에서 제공하는 error page는 <u>공식적인 서비스에 활용하기에 적합하지 않은 표현을 사용</u>하고 있습니다. 공식적인 서비스 사용시에는 꼭 에러 페이지들을 별도로 만드는 것을 권고합니다.

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
3. ***/etc/logrotate.d/nginx*** 에서 log ratation 설정을 하고 있습니다.
4. ***/etc/sysconfig/nginx***에서 init script에 필요한 옵션을 설정하고 있습니다.

##3. 안녕에 제공하는 추가 모듈

##4. PHP 연동
##5. JAVA/Python/Perl 연동

  안녕 리눅스에서 특별히 반영한 것이 없습니다. 기존에 하시던 방법이 있으면 그대로 하시면 무방 합니다. 또한, 인터넷 검색을 활용 하십시오. 

##6. nginx 구동

  간단한 apcahe control 방법에 대하여 기술 합니다.

  * 부팅시 nginx 시작하도록 설정
  ```bash
  [root@an3 ~]$ service httpd enable
  ```
  * 부팅시 nginx 시작 하지 않도록 설정
  ```bash
  [root@an3 ~]$ service httpd disable
  ```
  * nginx 시작
  ```bash
  [root@an3 ~]$ service httpd start
  ```
  * nginx 정지
  ```bash
  [root@an3 ~]$ service httpd stop
  ```
  * nginx 재시작
  ```bash
  [root@an3 ~]$ service httpd restart
  ```
  * nginx 상태 보기
  ```bash
  [root@an3 ~]$ service httpd status
  ```
