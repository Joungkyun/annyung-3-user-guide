# Chapter 5. DNS 운영

이 챕터에서는 안녕 리눅스 3을 이용하여 DNS 구성을 하는 것을 설명 합니다.

안녕 리눅스 3은 대부분 RHEL 7 또는 CentOS 7과 호환이 됩니다. 다만, ***bind*** 와 ***httpd***, ***php*** package는 ***호환이 되지를 않는데***, 이 챕터에서는 호환이 되지 않는 패키지 중 1개인 ***bind*** pcakge를 이용 합니다.

RHEL 7 또는 CentOS 7과 호환이 되지 않는 부분은 아래에 기술이 되니, RHEL 7 또는 CentOS 7 사용자도 이 부분만 참고를 하면 이 문서를 이용하여 DNS 구성을 하는데 무리가 없습니다.

이 문서는 HowTo 형식으로 기술이 됩니다. DNS에 대한 기본 이해는 있다는 가정하에 설명을 합니다.

## RHEL 7 / CentOS 7 과 다른 점

1. 안녕 리눅스의 bind package는 기본으로 ***chroot*** 가 적용이 되어 있습니다. 그러므로 모든 설정 파일은 ***/var/named*** 아래에 존재 합니다. ***chroot*** 하지 않는 bind와의 호환성을 위하여 ***named.conf***가 위치하는 ***/var/named/etc*** 는 ***/etc/named*** 로 soft link 처리 되어 있습니다.
2. ***named.conf*** 설정이 ***named.XXXXX.conf***와 같이 기능별로 설정 파일이 구분이 되어 있습니다. 이 의미는, ***named.XXXX.conf*** 에 들어갈 내용은 ***named.conf***에 들어가도 상관이 없다는 의미이며, ***named.conf***나 ***named.XXXXX.conf***에 들어갈 내용들은 ***named.conf*** 또는 어떤 ***named.XXXXX.conf*** 에 들어가도 상관이 없다는 의미입니다.


