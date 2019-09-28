# httpd

## Descriptions:

Apache HTTP Server

## Changes on AnNyung:

1. 2.4.18 업데이트
2. 기본 MPM : **event**
   * libphp7 을 사용하기 위해서는 prefork MPM으로 변경해야 함.
   * libphp7보다는 event MPM에서 php-fpm 사용을 권장.
3. mod\_http2 지원
4. httpd-conf package 분리
   * fancy icon 변경
   * 기본 error document 변경
5. ssi 변수에서 한글 깨지는 문제 수정
6. External404Iframe 지시자 추가 \(기본값: On\)
   * 404 page를 외부 URL로 지정했을 경우, 서버 응답코드가 200으로 처리되는 문제를 방지하기 위하여 404 error page에 iframe으로 외부 URL을 보여주는 기능

## Sub packages:

* **httpd-devel** - Development interfaces for the Apache HTTP server
* **httpd-manual** - Documentation for the Apache HTTP server
* **httpd-tools** - Tools for use with the Apache HTTP Server
* [**httpd-conf**](../annyung3-core-packages/pkg-core-httpd-conf.md) - 안녕 리눅스 아파치2의 바이너리를 제외한 설정파일들을 포함
* **mod\_ldap** - LDAP authentication modules for the Apache HTTP Server
* **mod\_proxy\_html** - HTML and XML content filters for the Apache HTTP Server
* **mod\_session** - Session interface for the Apache HTTP Server
* **mod\_ssl** - SSL/TLS module for the Apache HTTP Server
* [**httpd-url**](../annyung3-core-packages/pkg-core-httpd-url.md) - 아파치2 URL 모듈
* [**httpd-krisp**](../annyung3-core-packages/pkg-core-httpd-krisp.md) - 아파치2 KRISP 모듈
* [**httpd-nis**](../annyung3-core-packages/pkg-core-httpd-nis.md) - 아파치2 NIS 인증 모듈

