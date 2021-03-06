# lighttpd

## Description:

빠른 메모리 기반 웹 서버 \(lighttpd\)

## Features:

1. include 지시자에 **astrik** 사용 가능하도록 수정
   * [http://redmine.lighttpd.net/issues/1221](http://redmine.lighttpd.net/issues/1221)
   * 파일이 존재하지 않을 경우에도 error 처리 하지 않음
2. Apache style의 KeepAlive reponse header 지원
   * [http://redmine.lighttpd.net/issues/1284](http://redmine.lighttpd.net/issues/1284)
3. remote path의 사용자 404 에러 페이지 지원

   ```text
   server.error-handler-404 = "http://domain.com/errs/404.html"
   ```

4. TCP backlog를 설정 파일에서 변경 가능하도록 패치

   ```text
   server.backlog = 1024
   ```

5. mod\_dirlisting 기능 향상
   * dir-listing.gallery  
     * 디렉토리 listing에 이미지 파일이 있을 경우, &lt;img&gt; tag로 출력
     * 파일 이름이 cover\* 또는 preview-000 형식일 경우 cover mode로 출력
   * dir-listing.encoding
     * 문서의 charset을 지정
   * dir-listing.html-lang
     * &lt;html lang="VALUE"&gt;
   * dir-listing.urlencode
     * urlencoding 하여 출력 \(기본값: enable\)
   * dir-listing.external-js
     * listing시 지정한 외부 javascript를 삽입
6. 추가 모듈
   * [mod\_throttlestatus](http://svn.oops.org/wsvn/Lighttpd.mod_throttlestatus/trunk/throttlestatus.ko.txt)
   * [mod\_url](http://svn.oops.org/wsvn/Lighttpd.mod_url/trunk/README)
   * [mod\_net\_access](http://svn.oops.org/wsvn/Lighttpd.mod_net_access/trunk/README)
   * [mod\_auth\_nis](http://svn.oops.org/wsvn/Lighttpd.mod_auth_nis/trunk/README)
   * [mod\_krisp](http://svn.oops.org/wsvn/Lighttpd.mod_krisp/trunk/README)

## Reference:

* [http://redmine.lighttpd.net/projects/lighttpd/wiki](http://redmine.lighttpd.net/projects/lighttpd/wiki)

## Dependencies:

* [libimginfo](../annyung3-core-packages/pkg-core-libimginfo.md)

## Sub Packages:

* None

## Related Packages:

* [httpd](../annyung3-base-packages/pkg-base-httpd.md)
* [nginx](pkg-addon-nginx.md)

