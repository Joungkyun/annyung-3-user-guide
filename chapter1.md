# Chapter 1. 안녕 리눅스 3 \/ CentOS 7.2 차이점

## 1. 안녕 리눅스의 특징

1. 사용자 환경\(X windows 환경\)을 제거하고 compact 한 서버 전용 배포본
2. Enterprise 환경에서 검증된 배포본

   ```text
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
   * [kmod\_geoip](chapter2/chapter2-1-firewall/chapter2-1-firewall-6.md)
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
   * [PHP56](annyung3-package-catalog/annyung3-addon-packages/pkg-addon-php56.md)
   * [PHP71](annyung3-package-catalog/annyung3-addon-packages/pkg-addon-php71.md)
   * [libevent14](annyung3-package-catalog/annyung3-addon-packages/pkg-addon-libevent14.md)
   * [sqlite32](annyung3-package-catalog/annyung3-addon-packages/pkg-addon-sqlite32.md)
9. Oracle JVM 환경 지원

## 3. 보안 고도화

1. [chroot 환경 강화](chapter2/chapter2-2-pam-control/chapter2-2-pam-control-2.md) \([pam chroot](annyung3-package-catalog/annyung3-base-packages/pkg-base-pam.md) 모듈 개선\)
2. [ISMS](http://isms.kisa.or.kr/kor/main.jsp) 인증 [심사 기준 적용](chapter2/chapter2-2-pam-control/)
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
3. **PHP 7.1** compatible package 지원 \(php71-fpm\)
4. realpath\_cache\_force 지원 \(file system 탐색 성능 향상\)
5. * **open\_basedir** 사용시에 30% 정도 성능 향상
6. ZEND VM을 **GOTO mode로 빌드**하여 기본 CALL type VM보다 20% 성능 향상
7. Mariadb 10.1 업데이트

## 5. 차이점 일람

차이점 일람은 RHEL/CentOS 사용자가 안녕 리눅스로 전환할 때, 고려해야 할 점이나, 달라진 점을 정리 합니다.

### 5.1 OS banner

안녕 리눅스의 _**/etc/centos-release**_와 _**/etc/redhat-release**_는 수정되지 않기 때문에, 각종 application의 OS detect 시에 _**CentOS 7**_ 또는 _**RHEL 7**_로 인식이 되어집니다. 이 의미는 그만큼 호환이 가능하다는 의미입니다. 안녕 리눅스 배너는 _**/etc/annyung-release**_와 _**/etc/system-release**_, _**/etc/system-release-cpe**_ 에 적용이 되어 있으므로, 안녕 리눅스를 detect 하기 위해서는 이 파일들을 체크 해야 합니다.

### 5.2 방화벽

#### 5.2.1 방화벽 프로그램

안녕 리눅스는 _**RHEL/CentOS**_ 7의 _**firewalld**_ 대신 [_**oops-firewall**_](http://oops.org/?t=lecture&sb=firewall&n=2) 을 사용합니다. _**oops-firewall**_은 직관적이고 관리가 쉬우며, iptables를 잘 이해하고 있는 경우에는 직접 rule set을 조정할 수도 있습니다.

_**oops-firewll**_ 대신 _**firewalld**_를 사용하고 싶다면, 다음 명령으로 가능 합니다.

```bash
[root@an3 ~]$ service oops-firewall stop
[root@an3 ~]$ yum remove oops-firewall
[root@an3 ~]$ yum install firewalld
```

_**oops-firewall**_은 1999년 부터 개발이 되어온 iptables frontend 방화벽 관리자로서, 15년 이상 enterprise 환경에서 검증이 된 서버 방화벽 프로그램 입니다.

_**/etc/oops-firewall/filter.conf**_에서 서비스에 대한 설정을 할 수 있으며, 설치시에는 기본으로 22번 ssh port만 open 되어 있습니다.

_**oops-firewall**_은 기본적으로 /etc/oops-firewall 에 있는 설정 파일을 파싱하여 iptables ruleset을 생성하며, cmd 상에서 _**oops-firewall**_ 이라는 명령어를 실행하면 모든 rule set을 초기화 한 후에 새로운 rule set을 작성하기 때문에 복잡한 실행 옵션 같은 것이 없습니다.

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

_**oops-firewall**_에 대한 자세한 사항은 [http://oops.org/?t=lecture&sb=firewall&n=2](http://oops.org/?t=lecture&sb=firewall&n=2) 문서를 참고 하십시오.

#### 5.2.2 iptables + GeoIP 연동

안녕 리눅스는 기본적으로 iptables와 GeoIP가 연동이 되어 있어 국가별 rule set을 사용할 수 있습니다. _**RHEL/CentOS**_ 7에서 iptables 와 GeoIP 연동을 원한다면, [안녕 리눅스 3 repository](http://mirror.oops.org/pub/AnNyung/3/)에서 다음 패키지를 다운로드 받아서 업데이트 하면 사용할 수 있습니다.

* kmod-geoip
* iptables

```bash
[root@an3 ~]$ # 한국이 아닌 곳에서의 ssh 접근 방어
[root@an3 ~]$ iptables -A INPUT -p tcp --dport 22 -m geoip ! --src-cc KR -j DROP
```

또는 oops-firewall 에서는 _**/etc/oops-firewall/user.conf**_ 에서 사용 가능

```bash
[root@an3 ~]$ cat /etc/oops-firewall/user.conf
# 한국 외의 ssh 접근 거부
%-A INPUT -p tcp --dport 22 -m geoip ! --src-cc KR -j DROP
[root@an3 ~]$
```

### 5.3 Yum

### 5.3.1 AnNyung LInux repository

안녕 리눅스 repository 설정은 _**/etc/yum.repos.d/AnNyung.repo**_ 에 있습니다. 안녕 리눅스 repository 중 _**Base**_ repository는 _**RHEL/CentOS**_의 패키지를 수정한 다음, Epoch를 올려서 _**RHEL/CentOS**_ 원래의 패키지가 다시 원복되지 않도록 하고 있습니다. 안녕 리눅스 repository의 업데이트 상황은 [http://annyung.oops.org/?m=update&p=3](http://annyung.oops.org/?m=update&p=3) 에서 확인할 수 있으며, [RSS](http://annyung.oops.org/rss.php?v=3)를 제공하고 있습니다.

### 5.3.2 Package 업데이트

안녕 리눅스는 처음 설치 시에 _**CentOS/RHEL**_과는 달리 _**yum-cron**_ 패키지가 기본으로 설치가 됩니다. 이 의미는 1일 1회 패키지 체크를 통하여 패키지 업데이트를 자동 실행 한다는 의미입니다.

만약, 패키지 업데이트를 원하지 않는다면, 정말 권장하지는 않지만 다음의 명령으로 중지할 수 있습니다.

```bash
[root@an3 ~]$ service yum-cron stop
```

특정 package만 업데이트를 원하지 않을 경우에는 _**yum**_ 설정 파일에서 _**exclude**_ 옵션을 이용하여 설정 할 수 있습니다.

1. yum 전체 설정에 반영할 경우 \(_**/etc/yum.conf**_\)

   ```bash
   [root@an3 ~]$ cat /etc/yum.conf
   ...상략
   # 모든 i386/i686 package, GeoIP-data, libkrisp-data package를 yum으로 관리하지 않음
   exclude=*.i*86 GeoIP-data libkrisp-data
   ...하략
   ```

2. 특정 repository 에 반영할 경우

   ```bash
   [root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.repo
   ..상략
   [AN:base]
   name=AnNyung $annyungver Base Repository
   mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
   # 안녕리눅스의 bind를 사용하지 않고 RHEL의 bind를 사용할 경우
   exclude=bind*
   ..하략
   ```

### 5.3.3 X 기능 제거

안녕 리눅스는 서버 전용 배포본을 추구합니다. 그렇기 일부 패키지에서 X 기능이 제거된 경우가 있습니다. 이 패키지들은 안녕 리눅스의 [_**Xless**_ repository](http://mirror.oops.org/pub/AnNyung/3/xless/x86_64/)에서 관리 되고 있습니다. 만약 서버에 Oracle을 설치해야 하는 경우에는 Oracle installer가 X windows 환경을 요구하므로, _**Xless**_ repository를 disable 한 후에 X 관련 패키지를 설치해 주어야 합니다.

_**Xless**_ repository의 패키지들은 안녕 리눅스 처음 설치시에 기본으로 설치되지 않습니다.

```bash
[root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.reps
... 상략
[AN:xless]
name=AnNyung $anntyung-ver X less Repository
mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=xless
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
enabled=0
... 하략
```

### 5.4 계정 정책

#### 5.4.1 root login

1. 기본적으로 root login 제한
2. 사설\(private\) IP에서는 interactive root shell 접근 가능
   * "`ssh root@host.com`" 접근 제한
   * "`ssh root@host.com 'ls -al'`" 가능
3. /root/.bashrc 에서 설정 변경 가능. [rootfiles](https://joungkyun.gitbooks.io/annyung-3-user-guide/content/pkg-base-rootfiles.html) 패키지 일람 참조.

#### 5.4.2 account policy 변경

_**ISMS**_ 권고 사항 반영.

1. 암호 최소 길이 8자
2. 암호 생성 시, 대문자/소문자/숫자/특수문자 중 3개가 포함이 되어야 함
3. 이전 4개의 암호를 기억
4. 5회 로그인 실패 시, 120초간 계정 잠금
5. 암호 만료 90일\(3개월\).
   * _**/etc/login.defs.exception**_ 에 암호 만료 예외 account 설정
6. [Shell login Control \(with PAM\)](https://joungkyun.gitbooks.io/annyung-3-user-guide/content/chapter2-2-pam-control.html) 문서 참조

### 5.4 Shell

#### 5.4.1 한국어 출력

**jfbterm\*** 패키지가 기본 설치 되어 있어, local console에서 한글 출력이 기본으로 가능. \(현재 입력은 버그가 있어 안됨\).

```bash
[root@an3 ~]$ jfbterm
[root@an3 ~]$ cat utf8-hangul.txt
UTF-8 한글 입니다.
[root@ane ~]$ exit
```

#### 5.4.2 shell 추적

1. USER\_LANG 환경 변수를 가지고 있을 경우, ssh 접속시에나 sudo 전환 시에 LANG 환경 변수를 USER\_LANG 환경변수를 이용하여 상속.
   * 시스템 환경 변수 LANG=ko\_KR.UTF-8 일 경우
   * _**putty**_에 USER\_LANG="ko\_KR.eucKR" 환경 변수 설정
   * _**putty**_로 로그인시, LANG 변수 값은 _**ko\_KR.eucKR**_이 됨.
   * 이 상태에서 sudo를 했을 경우에도, _**ko\_KR.eucKR**_이 유지됨.
2. shell time out 이 600초로 기본 설정 됨.
3. sudo를 이용하여 account 전환을 했을 경우, 전환된 계정의 history에 원래의 user 기록.

   ```bash
   [root@an3 ~]$ history
   14  2016-02-03 16:18:01 oops make a && make s
   15  2016-02-03 16:18:53 oops ls
   16  2016-02-03 16:18:56 oops ./sync
   17  2016-02-03 16:20:12 oops vi /var/log/yum.log
   18  2016-02-03 16:37:22 james rpm -q openssl
   19  2016-02-03 16:38:06 james cd ../SPECS/
   20  2016-02-03 16:38:06 james ls
   21  2016-02-03 16:38:11 oops rpm -q openssh
   ```

### 5.5 CentOS/RHEL 7과 비호환 사항

_**httpd**_, _**php**_, _**bind**_ 의 경우는 _**RHEL/CentOS**_와 호환되지 않습니다. 설정 파일의 구성 등이 다르기 때문에 이 3개의 패키지의 경우에는 기존의 설정 파일을 사용할 수 없고, 처음 부터 설정을 다시 하여야 합니다.

_**httpd**_, _**php**_의 경우에는 [3. Web Control](https://joungkyun.gitbooks.io/annyung-3-user-guide/content/chapter3.html)문서를 참고 하십시오. _**fastcgi**_를 이용하여 _**php56**_ 또는 _**php71**_을 사용할 수 있습니다.

_**bind**_의 경우에는 기본으로 chroot기능으로 packaging 되어 있으므로, _**/var/named**_ 에 모든 설정이 들어가게 됩니다. _**/etc/named**_ 는 설정 호환을 위하여 soft link로 지원이 됩니다.

## 6. 기타

위의 특징들 외에 많은 변경 사항이 있으며, 이에 대해서는 "[안녕 리눅스 3 패키지 일람](annyung3-package-catalog/)"을 참조 하십시오.

