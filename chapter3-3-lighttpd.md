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



##2. ningx 설정 파일





##3. SSL 설정 및 HTTP2 protocol 지원



##4. 안녕에서 제공하는 추가 모듈



##5. PHP 연동



##6. JAVA(tomcat)/Python/Perl 연동

  안녕 리눅스에서 특별히 반영한 것이 없습니다. 기존에 하시던 방법이 있으면 그대로 하시면 무방 합니다. 또한, 인터넷 검색을 활용 하십시오. 

##7. nginx 구동

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
