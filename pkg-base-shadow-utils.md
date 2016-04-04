# shadow-utils

### Description:
Utilities for managing accounts and shadow password files

### Changes on AnNyung:
1. [ISMS](http://isms.kisa.or.kr/kor/intro/intro01.jsp) 인증 지원
 * password 최소 길이 5자 => 8자로 상향
 * password 만료일을 99999일에서 90일로 하향
 * password 만료 예외 리스트 추가 (_/etc/login.defs.exception_)
 * [Shell login Control (with PAM)](chapter2-2-pam-control.md) 문서 참조
2. pwconv/pwunconv -l 옵션 추가
 * *-l* 옵션으로 지정된 directory에 있는 passwd/shadow 파일 관리 가능
  ```bash
  [root@an3 ~]$ mkdir /var/tmp/etc
  [root@an3 ~]$ cp -af /etc/passwd /etc/shadow /var/tmp/etc
  [root@an3 ~]$ pwunconv -l /var/tmp/etc
  [root@an3 ~]$ pwconv -l /var/tmp/etc
  ```
 * *-l* 옵션이 지원하지 않는다는 메시지가 나오면, ***yum update***를 해 주십시오.

### Sub packages:
