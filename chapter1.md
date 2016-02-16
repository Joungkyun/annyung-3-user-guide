# Chapter 1. 안녕 리눅스 3 / CentOS 7.2 차이점

## 1. 안녕 리눅스의 특징

 1. 사용자 환경(X windows 환경)을 제거하고 compact한 서버 전용 배포본
 2. Enterprise 환경에서 검증된 배포본

## 2. 운영 고도화
  1. 콘솔 한글 출력 지원 (jfbterm)
  2. IP 기반 GEO system 지원
    * kmod_geoip
    * iptables xt_geoip
    * libkrisp
  3. cvs usermap 기능 지원으로 공용 repository 운영 고도화
  4. IDN 지원 (bind, ssh client, whois 등등)
  5. rsyslog mysql backend에서 mysql unix domain socket 사용 가능
  6. tcping, tcptraceroute 등 ICMP 제한된 네트워크 탐지를 위해 기본 제공
  7. vim
    * PHP native manual 지원(shift + K)
    * vim folder 기능 개선
    * checksyntax 플러그인 추가
  8. legacy 호환 패키지 지원
    * [PHP56](pkg-addon-php56.md)
    * [libevent14](pkg-addon-libevent14.md)
    * [sqlite32](pkg-addon-sqlite32.md)
  9. Oracle JVM 환경 지원


## 3. 보안 고도화
  1. chroot 환경 강화 ([pam chroot](pkg_base_pam.md) 모듈 개선)
  2. [ISMS](http://isms.kisa.or.kr/kor/main.jsp) 인증 심사 기준 적용
  3. account action 추적 시스템
    * su, sudo 시에 SU_USER 환경 변수에 origianl account 유지
    * history에 SU_USER 반영
  4. PHP
    * PHP shell injection 원천 방지 (exec_dir)
    * 파일 업로드시, image header에 injection code 존재 여부 탐지
    * exec_dir과 open_basedir을 이용하여 remote 접근 환경 제한 가능
    * 기본적으로 .php 확장자만 php compile이 가능하도록 제한
  5. bind chroot 환경을 default로 변경
  6. 1일 1회 yum update 기본 작동 (yum-cron: RHEL/CentOS는 기본으로 동작 안함)


## 4. 서비스 고도화
  1. HTTP2 protocol 지원 (ALPN 지원)
    * http 2.4.18 mod_http2 (안녕 기본 지원)
    * nginx 1.9.5 http2 module 가능 (안녕 3에서는 stables인 1.8 지원)
  2. PHP 고도화
    * PHP 7 support
    * PHP 5.6 compatible package 지원 (php-fpm)
      * PHP 5.3 copatible mode 지원
    * realpath_cache_force 지원 (file system 탐색 성능 향상)
    * ZEND VM을 GOTO mode로 빌드하여 기본 CALL type VM보다 15% 성능 향상
  3. Mariadb 10.1 업데이트

## 5. 기타

위의 특징들 외에 많은 변경 사항이 있으며, 이에 대해서는 "[안녕 리눅스 3 패키지 일람](AnNyung3-Package-Catalog.md)"을 참조 하십시오.