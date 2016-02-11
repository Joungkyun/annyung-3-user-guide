# lighttpd

### Description:
빠른 메모리 기반 웹 서버 (lighttpd)

### Features:
1. include 지시자에 **astrik** 사용 가능하도록 수정
 * http://redmine.lighttpd.net/issues/1221
 * 파일이 존재하지 않을 경우에도 error 처리 하지 않음
2. Apache style의 KeepAlive reponse header 지원
 * http://redmine.lighttpd.net/issues/1284
3. remote path의 사용자 404 에러 페이지 지원
 ```ini
 server.error-handler-404 = "http://domain.com/errs/404.html"
 ```
4. TCP backlog를 설정 파일에서 변경 가능하도록 패치
 ```ini
 server.backlog = 1024
 ```
5. mod_dirlisting 기능 향상
 * gallery 기능 지원
 * dir-listing.encoding 추가
 * dir-listing.html-lang 추가
6. 추가 모듈
 * mod_throttlestatus
 * mod_url
 * mod_net_access
 * mod_auth_nis
 * mod_krisp

### Reference:
* None

### Dependencies:
* [libimginfo](pkg-core-libimginfo.md)

### Sub Packages:
* None

### Releated Packages:
* None