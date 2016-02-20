# shadow-utils

### Description:
Utilities for managing accounts and shadow password files

### Changes on AnNyung:
1. [ISMS](http://isms.kisa.or.kr/kor/intro/intro01.jsp) 인증 지원
 * password 최소 길이 5자 => 8자로 상향
 * password 만료일을 99999일에서 90일로 하향
 * password 만료 예외 리스트 추가 (_/etc/login.defs.exception_)
 * [Shell login Control (with PAM)](chapter2-2-pam-control.md) 문서 참조

### Sub packages:
