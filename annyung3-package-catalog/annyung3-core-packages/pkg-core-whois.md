# whois

## Description:

whois/nicname 클라이언트 프로그램

## Features:

1. recursion 지원
2. IDN 지원

## Reference:

* [http://svn.oops.org/wsvn/OOPS.kwhois/trunk/](http://svn.oops.org/wsvn/OOPS.kwhois/trunk/)

  ```bash
  [root@an3 x86_64]$ whois -h
  사용법: whois [OPTIONS...] 질의[@서버[:포트]]
  유효한 옵션들:
       -h 서버    whois 서버 이름
       -p 포트    서버 포트
       -t 초      질의 시간 제한
       -r         반복을 강행
       -n         반복 하지 않음
       -v         디버그 모드
       --         쿼리의 일부분으로 취급됨
  기본 서버: whois.verisign-grs.com
  kwhois 4.4
  [root@an3 x86_64]$
  ```

## Dependencies:

* None

## Sub Packages:

* None

## Releated Packages:

* None

