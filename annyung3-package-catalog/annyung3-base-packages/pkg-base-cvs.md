# cvs

## Descriptions:

Concurrent Versions System

## Changes on AnNyung:

1. usermap 기능 추가
   * login user를 다른 권한의 user로 mapping
   * CVSROOT/usermap 에서 관리
   * "**LOGIN\_USER**:**MAPPING\_USER**" 형식으로 관리
2. --deault-usermap 옵션 추가
   * usermap이 없더라도 강제로 login user를 다른 user로 mapping
3. 1.12 branch에서 rls 기능 backport
4. xinetd 서버 설정 파일 추가
   * _**/etc/xinetd.d/cvs**_ \(cvs-inetd package\)

## Sub packages:

* **cvs-contrib** - Unsupported contributions collected by CVS developers
* **cvs-doc** - Additional documentation for Concurrent Versions System
* **cvs-inetd** - CVS server configuration for xinetd

