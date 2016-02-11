안녕 리눅스 3 사용자 가이드
=======

## 1. 개요

안녕 리눅스는 Redhat 7.2를 기반으로 사용자 사용 환경(User interface)를 제거한 서버 전용의 작고 가벼운 배포본으로 2003년 6월 13일에 1.0이 release가 되었습니다.

1.0 release 이후, 안녕 리눅스는 Rehat 배포본을 탈피하기 위한 방향으로 진행을 하였으나, 1인 프로젝트의 한계로 kernel 2.6 버전 도입이 늦어지고 관리상의 버거움이 발생하여, RHEL/CentOS 의 minimal version에 안녕 리눅스의 특징만을 변경한 repository를 운영하는 방식으로 선회를 하여 2012년 6월 RHEL 6u2/CentOS 6.2를 기반으로 2를 릴리즈 하게 되었습니다.

그리고, 2013년 2월, RHEL 7u2/CentOS 7.2를 기반으로 안녕 리눅스 3을 릴리즈 하게 되었습니다.

안녕 리눅스 최초의 시작은 비대해지는 Redhat 배포본이 너무 버거워 작고 가벼운 배포본을 만들자는 것이 목표였으며, 2.0 이후로는 서버 전용의 enterprise 배포본에 중점을 두고 있습니다. 안녕 리눅스는 CDN  업체 W사의 CDN Game publishing/patch download server(리니지, 마비노기등등의 대형 게임), Game portal N사, 전자상거래 업체 T사의 core system, KLDP에서 주 OS로 사용되어 enterprise 환경에서 검증이 되었고 knowhow를 가지고 있는 배포본 입니다.


## 2. 안녕 리눅스의 특징 / CentOS 7과의 차이점

1. 사용자 환경을 제외한 compact한 서버 전용 배포본
2. Enterprise 환경에서 검증된 배포본
3. ISMS 인증을 위한 deploy 반영
4. 한글 환경을 위한 지원
5. 보안성 강화
6. EPEL repository 기본 지원
7. 1일 1회 package 자동 업데이트(yum-cron 동작)
