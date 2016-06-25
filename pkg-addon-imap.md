# imap

### Description:
IMAP 과 POP 프로토콜 서버 데몬

### Features:
1. protocol 별 xinetd 구동 package 분리

### Reference:
* http://www.washington.edu/imap/

### Dependencies:
* None

### Sub Packages:
* **imap-devel** - Imap 라이브러리를 사용하여 프로그래밍을 하기 위한 개발 툴
* **imap-doc** - imap 문서
* **imap-init-imap2** - imap v2 프토토콜 구동 파일 (with xinetd)
* **imap-init-imaps** - 암호화된 imap 프로토콜 구동 파일 (with xinetd)
* **imap-init-pop3** - pop3 프로토콜 구동 파일 (with xinetd)
* **imap-init-pop3s** - 암호화된 pop3 프로토콜 구동 파일 (with xinetd)

### Related Packages:
* [xinted](pkg-base-xinetd.md)