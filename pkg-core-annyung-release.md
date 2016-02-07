# annyung-release

### Description:

annyung-release 패키지는 안녕 리눅스 3의 릴리즈 정보를 가지고 있다

### Features:

1. 안녕 리눅스 banner 구성
  * /etc/issue[.net]
  * /etc/annyung-release
  * /etc/system-release[-cpe]
  * /etc/os-release
  * /etc/grub2.cfg
  * RHEL/CentOS compat (Not include)
    * /etc/redhat-release
    * /etc/centos-release<br><br>

2. 안녕 리눅스 구성 유지를 위한 cronjob
  1. 파일 경로: /etc/cron.d/annyung-banner
  2. issue/issue.net의 안녕 리눅스 배너 유지
  3. os-release 안녕 리눅스 배너 유지
  4. Ethernet device이름을 eth[0-9]로 유지
  5. fstab에 disk 장치에 대해서 noatime을 유지
  6. wheel 그룹에 root 추가