# bind

## Descriptions:

Chroot가 적용된 버클리 인터넷 네임 서버 \(BIND\)

## Changes on AnNyung:

1. bind-chroot 패키지 제거하고, 기본으로 chroot로 동작하도록 구성 \(_/var/named_\)
2. _/etc/named/naemd.conf_는 _/var/named/etc/named.conf_의 solft link \(호환성 유지\)
3. zone file과 _named.conf_에서 IDN을 직접 사용 가능
   * [http://annyung.oops.org/?m=white&p=mdns](http://annyung.oops.org/?m=white&p=mdns)
4. multiple CNAME 지원
5. geodns 기능 지원
   * _/etc/sysconfig/named_에 GEOIP\_DATA\_COPY="yes" 설정
   * [https://code.google.com/p/bind-geoip/wiki/UsageGuide](https://code.google.com/p/bind-geoip/wiki/UsageGuide) 참조
   * _**RHEL/CentOS**_ 7.3에서 추가된 _**GeoDNS**_ 기능과 다름!!
     * 안녕 리눅스는 _**RHEL/CentOS**_ 7.3과는 다른 패치로 지원을 하며, _**RHEL/CentSO**_ 보다 먼저 지원을 시작한 이유로 이전 버전과의 호환성을 위하여 _**RHEL/CentOS**_과는 다른 기능으로 제공
   * 안녕 리눅스 3.5 부터는 _**RHEL/CentOS**_ 7.3 에 추가된 _**GeoDNS**_ 를 사용합니다. 주의 하십시오. 사용 방법은 [https://kb.isc.org/docs/aa-01149](https://kb.isc.org/docs/aa-01149) 문서를 참고 하십시오.

## Notice:

DNSSEC와 database backend\(bind-sdb\)에 관련된 테스트는 안녕에서 보증하지 않습니다. 이 기능을 사용하기 위해서는 CentOS 7의 bind package를 사용하십시오.

1. _/etc/yum.repos.d/AnNyung.repo_에서 \[AN:base\] 섹션에 **"exclude=bind\*"** 추가

   ```text
   [AN:base]
   name=AnNyung $annyungver Base Repository
   mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
   exclude= bind*
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
   ```

2. 설치되어 있는 bind package 확인

   ```bash
   [root@an3 ~]$ rpm -qa | grep bind
   bind-license-9.9.4-29.an3.2.noarch
   bind-utils-9.9.4-29.an3.2.x86_64
   bind-libs-9.9.4-29.an3.2.x86_64
   bind-libs-lite-9.9.4-29.an3.2.x86_64
   [root@an3 ~]$
   ```

3. CentOS bind로 교체

   ```bash
   [root@an3 ~]$ yum install -y yum-utils
   [root@an3 shm]$ yumdownloader bind-utils bind-libs bind-libs-lite bind-license
   Loaded plugins: fastestmirror
   AN:addon                                          | 2.9 kB  00:00:00
   AN:base                                           | 2.9 kB  00:00:00
   AN:core                                           | 2.9 kB  00:00:00
   AN:xless                                          | 2.9 kB  00:00:00
   Loading mirror speeds from cached hostfile
        + AN:addon: mirror.oops.org
        + AN:base: mirror.oops.org
        + AN:core: mirror.oops.org
        + AN:xless: mirror.oops.org
        + base: ftp.kaist.ac.kr
        + epel: ftp.kddilabs.jp
        + extras: ftp.kaist.ac.kr
        + updates: ftp.kaist.ac.kr
   (1/4): bind-license-9.9.4-29.el7_2.2.noarch.rpm   |  82 kB  00:00:00
   (2/4): bind-libs-lite-9.9.4-29.el7_2.2.x86_64.rpm | 724 kB  00:00:00
   (3/4): bind-utils-9.9.4-29.el7_2.2.x86_64.rpm     | 200 kB  00:00:00
   (4/4): bind-libs-9.9.4-29.el7_2.2.x86_64.rpm      | 1.0 MB  00:00:00
   [root@an3 shm]$ ls
   bind-libs-9.9.4-29.el7_2.2.x86_64.rpm       bind-license-9.9.4-29.el7_2.2.noarch.rpm
   bind-libs-lite-9.9.4-29.el7_2.2.x86_64.rpm  bind-utils-9.9.4-29.el7_2.2.x86_64.rpm
   [root@an3 shm]$ rpm -Uhv bind-* --force
   ```

## Dependencies:

* [GeoIP](pkg-base-geoip.md)

## Sub packages:

* **bind-devel**- BIND DNS 개발을 위해 필요한 헤더 파일과 라이브러리
* **bind-libs** - BIND DNS에 필요한 라이브러리
* **bind-libs-lite** - DNS 프로토콜 동작을 위한 라이브러리
* **bind-license** - BIND DNS 라이센스
* **bind-lite-devel** - BIND DNS 개발을 위해 필요한 최소 헤더파일 및 라이브러리
* **bind-pkcs11** - 암호화를 위한 BIND 내장 PKCS\#11 기능
* **bind-pkcs11-devel** - Development files for Bind libraries compiled with native PKCS\#11
* **bind-pkcs11-libs** - Bind libraries compiled with native PKCS\#11
* **bind-pkcs11-utils** - DNSSEC를 사용하기 위한 BIND 내장 PKCS\#11 도구
* **bind-sdb** - 데이터베이스 백엔드와 DLZ를 지원하는 BIND 서버
* **bind-utils** - DNS 서버에 질의를 하기 위한 도구

