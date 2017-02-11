# Chapter 5. DNS 운영

이 챕터에서는 안녕 리눅스 3을 이용하여 DNS 구성을 하는 것을 설명 합니다.

안녕 리눅스 3은 대부분 RHEL 7 또는 CentOS 7과 호환이 됩니다. 다만, _**bind**_ 와 _**httpd**_, _**php**_ package는 _**호환이 되지를 않는데**_, 이 챕터에서는 호환이 되지 않는 패키지 중 1개인 _**bind**_ pcakge를 이용 합니다.

RHEL 7 또는 CentOS 7과 호환이 되지 않는 부분은 아래에 기술이 되니, RHEL 7 또는 CentOS 7 사용자도 이 부분만 참고를 하면 이 문서를 이용하여 DNS 구성을 하는데 무리가 없습니다.

이 문서는 HowTo 형식으로 기술이 됩니다. DNS에 대한 기본 이해는 있다는 가정하에 설명을 합니다.

## RHEL 7 / CentOS 7 과 다른 점

1. 안녕 리눅스의 bind는 기본으로 chroot가 적용이 되어 있으며, 모든 파일들은 ***/var/named*** 에 있습니다.  

   ```bash
   [root@an3 ~]$ cd /var/named/
   [root@an3 named]$ pwd
   /var/named
   [root@an3 named]$ ls
   dev  etc  log  run  usr  zone
   [root@an3 named]$
   ```

   ***/var/named*** 아래에 있는 디렉토리의 용도는 다음과 같습니다.

   * ***dev*** : bind 구동시에 필요한 device file들이 위치 합니다. 이는 chroot 환경 때문에 존재 합니다.
   * ***etc*** : ***named.conf***, ***rndc.conf*** 등의 bind 설정 파일들이 위치 합니다. 이 디렉토리는 ***/etc/named*** 로 soft link 되어 있습니다.
   * ***log*** : bind log가 저장이 됩니다. 역시 chroot 환경 때문에 필요하며, ***/var/log/named***로 soft link 되어 있습니다.
   * ***run*** : bind pid file이 저장 됩니다. chroot 환경 때문에 필요하며, ***/run/named*** 로 soft link 되어 있습니다.
   * ***usr*** : bind 구동시에 필요한 library들이 있습니다. 이는 chroot 환경 때문에 존재 합니다.
   * ***zone*** : DNS 추가 시에 필요한 zone file들이 위치 합니다.<br><br>

   즉, DNS 구성을 위해서는 ***/var/named/etc/*** 와 ***/var/named/zone*** 만 이용을 하며, 로그 확인 시에 ***/var/named/log*** 또는 ***/var/log/named*** 를 이용하시면 됩니다.

2. 모든 설정 파일은 _**/var/named/etc**_ 아래에 존재 합니다.

   ```bash
   [root@an3 etc]$ cd /var/named/etc/
   [root@an3 etc]$ ls
   named.acl.conf  named.iscdlv.key     named.root.key    pki        rndc.key
   named.conf      named.rfc1912.zones  named.user.zones  rndc.conf
   [root@an3 etc]$
   ```

  _**chroot**_ 하지 않는 bind와의 호환성을 위하여 _**named.conf**_가 위치하는 _**/var/named/etc**_ 는 _**/etc/named**_ 로 soft link 처리 되어 있습니다.

   ```bash
   [root@an3 etc]$ ls -l /etc/named
   lrwxrwxrwx 1 root named 16  2월 10 19:53 /etc/named -> ../var/named/etc
   [root@an3 etc]$
   ```

3. _**named.conf**_ 설정은 기능별로 아래와 같이 구분이 되어 있습니다. \(_**named.conf**_에서 아래의 순서대로 include 합니다.\)

   * _**named.acl.conf**_ : bind에서 사용할 각종 ACL group을 설정 합니다. 기본으로 _**LocalAllow**_ group이 설정 되어 있습니다.
   * _**rndc.conf**_ : _**rndc**_ 설정과 _**rndc key**_ 설정이 있습니다. 설치시에 기본으로 rndc key가 생성이 되므로, 굳이 따로 설정을 할 필요는 없습니다.
   * _**named.rfc1912.zones**_ : loopback 및 기본으로 필요한 zone 설정을 가지고 있습니다. 역시 건드릴 필요는 없습니다.
   * _**named.user.zones**_ : 사용자 domain 설정을 추가 합니다.


## BIND 설정

_**현재 문서 작성 중 입니다. - 2017/02/11 -**_

안녕 리눅스의 bind package는 설치 시에 기본적으로 구동이 되는데 문제가 없도록 기본 설정이 되어 있습니다. 하지만, 기본으로 localhost에서의 query만 허락하도록 되어 있으므로, DNS 운영 설정을 하기 위해서는 먼저 Recursion 정책을 결정하여 설정을 변경 해야 합니다.

1. [Recursion 설정](chapter5-1-recursion.md)
2. [Domain 추가](chapter5-2-add-domain.md)
3. [Slave DNS 구성](chapter5-3-slave-dns.md)
4. [DNSSEC 설정](chapter5-4-dnssec.md)
5. [GeoDNS 설정](chapter5-5-geodns.md)



