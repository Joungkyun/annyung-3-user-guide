# Chapter 1. 안녕 리눅스 3 / CentOS 7.2 차이점

## 1. 안녕 리눅스의 특징

1. 사용자 환경\(X windows 환경\)을 제거하고 compact 한 서버 전용 배포본
2. Enterprise 환경에서 검증된 배포본

   ```
   필자주>
   커널이나 glibc등의 주요 기능을 수정한 것이 없고, 대부분 PHP를
   제외하고는 성능과는 관계 없는 개선 이므로 딱히 검증이고 자시고 할
   이유도 없을 것 같습니다. 필자 견해로는 안녕 리눅스가 엔터프라이즈
   환경에 검증이 안되었다는 것은 CentOS가 검증이 안되었다는 것과 
   비슷한 견해가 아닐까 싶습니다. :-)

   일단 검증이 되었다고 하는 이유중에 하나는, enterprise 환경에서
   필자가 10여년 동안 직접 구축 운영을 하였고 현재도 사용하는 곳이 있기
   때문에 검증이라는 단어를 사용하였습니다.
   ```

## 2. 운영 고도화

1. 콘솔 한글 출력 지원 \(jfbterm\)
2. IP 기반 GEO system 지원

   * [kmod\_geoip](chapter2-1-firewall-6.md)
   * iptables xt\_geoip
   * libkrisp

3. cvs usermap 기능 지원으로 공용 repository 운영 고도화

4. IDN 지원 \(bind, ssh client, whois 등등\)

5. rsyslog mysql backend에서 mysql unix domain socket 사용 가능

6. tcping, tcptraceroute 등 ICMP 제한된 네트워크 탐지를 위해 기본 제공

7. vim

   * PHP native manual 지원\(shift + K\)
   * vim folder 기능 개선
   * checksyntax 플러그인 추가

8. legacy 호환 패키지 지원

   * [PHP56](pkg-addon-php56.md)
   * [PHP71](pkg-addon-php71.md)
   * [libevent14](pkg-addon-libevent14.md)
   * [sqlite32](pkg-addon-sqlite32.md)

9. Oracle JVM 환경 지원

## 3. 보안 고도화

1. [chroot 환경 강화](chapter2-2-pam-control-2.md) \([pam chroot](pkg-base-pam.md) 모듈 개선\)
2. [ISMS](http://isms.kisa.or.kr/kor/main.jsp) 인증 [심사 기준 적용](chapter2-2-pam-control.md)
3. account action 추적 시스템

   * su, sudo 시에 SU\_USER 환경 변수에 origianl account 유지
   * history에 SU\_USER 반영

4. PHP

   * PHP shell injection 원천 방지 \(exec\_dir\)
   * 파일 업로드시, image header에 injection code 존재 여부 탐지
   * exec\_dir과 open\_basedir을 이용하여 remote 접근 환경 제한 가능
   * 기본적으로 .php 확장자만 php compile이 가능하도록 제한

5. bind chroot 환경을 default로 변경

6. 1일 1회 yum update 기본 작동 \(yum-cron: RHEL/CentOS는 기본으로 동작 안함\)

## 4. 서비스 고도화

1. HTTP/2 protocol 지원 \(ALPN 지원\)

   * http &gt; 2.4.18 mod\_http2 \(안녕 기본 지원\)
   * nginx 1.9 http2 module
   * RHEL 7과 CentOS 7의 openssl 1.0.1e는 **ALPN**을 지원하지 않아 HTTP/2 지원을 못하지만, 안녕 3에서는 1.0.2의 **ALPN** 기능을 backporting 하여 지원 가능

2. PHP 고도화

   * **PHP 7** support
   * **PHP 5.6** compatible package 지원 \(php56-fpm\)

     * PHP 5.3 compatible mode 지원

* **PHP 7.1** compatible package 지원 \(php71-fpm\)

* realpath\_cache\_force 지원 \(file system 탐색 성능 향상\)

* * **open\_basedir** 사용시에 30% 정도 성능 향상

* ZEND VM을 **GOTO mode로 빌드**하여 기본 CALL type VM보다 20% 성능 향상

* Mariadb 10.1 업데이트


## 5. 차이점 일람

차이점 일람은 RHEL/CentOS 사용자가 안녕 리눅스로 전환할 때, 고려해야 할 점이나, 달라진 점을 정리 합니다.
***이 단락은 현재 작성 중 입니다.***


### 5.1 방화벽

#### 5.1.1 방화벽 프로그램

안녕 리눅스는 ***RHEL/CentOS*** 7의 ***firewalld*** 대신 [***oops-firewall***](http://oops.org/?t=lecture&sb=firewall&n=2) 을 사용합니다. ***oops-firewall***은 직관적이고 관리가 쉬우며, iptables를 잘 이해하고 있는 경우에는 직접 rule set을 조정할 수도 있습니다.

***oops-firewall***은 1999년 부터 개발이 되어온 iptables frontend 방화벽 관리자로서, 15년 이상 enterprise 환경에서 검증이 된 서버 방화벽 프로그램 입니다.

***/etc/oops-firewall/filter.conf***에서 서비스에 대한 설정을 할 수 있으며, 설치시에는 기본으로 22번 ssh port만 open 되어 있습니다.

***oops-firewall***은 기본적으로 /etc/oops-firewall 에 있는 설정 파일을 파싱하여 iptables ruleset을 생성하며, cmd 상에서 ***oops-firewall*** 이라는 명령어를 실행하면 모든 rule set을 초기화 한 후에 새로운 rule set을 작성하기 때문에 복잡한 실행 옵션 같은 것이 없습니다.

```bash
[root@an3 ~]$ # oops-firewall 실행
[root@an3 ~]$ oops-firwall
[root@an3 ~]$ # oops-firewall 실행 시에 처리 과정을 출력
[root@an3 ~]$ oops-firewall -v
[root@an3 ~]$ # 실제 실행을 하지는 않고, 실행될 rule set을 미리 보기
[root@an3 ~]$ oops-firewall -v -n
```

방화벽을 내리고 싶을 경우에는 다음의 systemd를 이용할 수 있습니다.

```bash
[root@an3 ~]$ service oops-firewall stop
```

***oops-firewall***에 대한 자세한 사항은 http://oops.org/?t=lecture&sb=firewall&n=2 문서를 참고 하십시오.

#### 5.1.2 iptables + GeoIP 연동

안녕 리눅스는 기본적으로 iptables와 GeoIP가 연동이 되어 있어 국가별 rule set을 사용할 수 있습니다. ***RHEL/CentOS*** 7에서 iptables 와 GeoIP 연동을 원한다면, [안녕 리눅스 3 repository](http://mirror.oops.org/pub/AnNyung/3/)에서 다음 패키지를 다운로드 받아서 업데이트 하면 사용할 수 있습니다.

 * kmod-geoip
 * iptables
 
```bash
[root@an3 ~]$ # 한국이 아닌 곳에서의 ssh 접근 방어
[root@an3 ~]$ iptables -A INPUT -p tcp --dport 22 -m geoip ! --src-cc KR -j DROP
```

또는 oops-firewall 에서는 ***/etc/oops-firewall/user.conf*** 에서 사용 가능

```bash
[root@an3 ~]$ cat /etc/oops-firewall/user.conf
# 한국 외의 ssh 접근 거부
%-A INPUT -p tcp --dport 22 -m geoip ! --src-cc KR -j DROP
[root@an3 ~]$
```


## 6. 기타

위의 특징들 외에 많은 변경 사항이 있으며, 이에 대해서는 "[안녕 리눅스 3 패키지 일람](AnNyung3-Package-Catalog.md)"을 참조 하십시오.




