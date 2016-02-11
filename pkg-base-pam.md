# pam

### Description:
응용프로그램에서 사용할 수 있는 인증을 제공하는 확장 라이브러리

### Changes on AnNyung:
1. chroot module 수정
 * group 설정 지원 (_%gruop_)
 * reverse 설정 지원 (_!user_)
 * 정규식 지원 (_/^user/_)
 * 예외 지원 (_-user_)
2. _/etc/login.def.exceptioin_
 * password expire를 하지 않을 예외 account 관리

### Sub packages:
* **pam-devel** - PAM을 이용한 PAM관련 응용프로그램 및 모듈 개발을 위해 필요한 파일들