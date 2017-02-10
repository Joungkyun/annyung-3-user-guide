# Chapter 5. DNS 운영

이 챕터에서는 안녕 리눅스 3을 이용하여 DNS 구성을 하는 것을 설명 합니다.

안녕 리눅스 3은 대부분 RHEL 7 또는 CentOS 7과 호환이 됩니다. 다만, ***bind*** 와 ***httpd***, ***php*** package는 ***호환이 되지를 않는데***, 이 챕터에서는 호환이 되지 않는 패키지 중 1개인 ***bind*** pcakge를 이용 합니다.

RHEL 7 또는 CentOS 7과 호환이 되지 않는 부분은 아래에 기술이 되니, RHEL 7 또는 CentOS 7 사용자도 이 부분만 참고를 하면 이 문서를 이용하여 DNS 구성을 하는데 무리가 없습니다.

이 문서는 HowTo 형식으로 기술이 됩니다. DNS에 대한 기본 이해는 있다는 가정하에 설명을 합니다.

## RHEL 7 / CentOS 7 과 다른 점

1. 안녕 리눅스의 bind package는 기본으로 ***chroot*** 가 적용이 되어 있습니다.  

   * 모든 설정 파일은 ***/var/named*** 아래에 존재 합니다.  
   ```bash
[root@an3 etc]$ cd /var/named/etc/
[root@an3 etc]$ ls
named.acl.conf named.rfc1912.zones pki
named.conf named.root.key rndc.conf
named.iscdlv.key named.user.zones rndc.key
[root@an3 etc]$
```

   * ***chroot*** 하지 않는 bind와의 호환성을 위하여 ***named.conf***가 위치하는 ***/var/named/etc*** 는 ***/etc/named*** 로 soft link 처리 되어 있습니다.  
   
2. ***named.conf*** 설정은 기능별로 아래와 같이 구분이 되어 있습니다. (***named.conf***에서 아래의 순서대로 include 합니다.)  
   
   * ***named.acl.conf*** : bind에서 사용할 각종 ACL group을 설정 합니다. 기본으로 ***LocalAllow*** group이 설정 되어 있습니다.
   * ***rndc.conf*** : ***rndc*** 설정과 ***rndc key*** 설정이 있습니다. 설치시에 기본으로 rndc key가 생성이 되므로, 굳이 따로 설정을 할 필요는 없습니다.
   * ***named.rfc1912.zones*** : loopback 및 기본으로 필요한 zone 설정을 가지고 있습니다. 역시 건드릴 필요는 없습니다.
   * ***named.suer.zones*** : 사용자 domain 설정을 추가 합니다.



