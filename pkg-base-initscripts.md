# initscrtips

### Description:
inittab 파일과 /etc/init.d 스크립트

### Changes on AnNyung:
 1. USER_LANG 환경 변수가 설정 되면, LANG 환경 변수를 USER_LANG 값으로 변경
  * _/etc/profile.d/lang.sh_
 2. changed sysctl value (/etc/sysctl.d/60-annyung.conf)
 3. patch init wrapper script on /sbin/service
  * init script 검색 대상에 _/sbin/init.d_ 추가

### Sub packages:
* **debugmode** - Scripts for running in debugging mode
