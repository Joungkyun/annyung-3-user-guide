안녕 리눅스 3 사용자 가이드
=======

Copyright &copy; 2016 [JoungKyun.Kim](https://oops.org/) all rights reserved.

## 1. 이력

* 2016.02.29 초판 발행

## 2. 개요

이 문서는 안녕 리눅스 3를 운영 관리하기 위한 정보를 전달 합니다.

기본적으로 안녕 리눅스는 CentOS 7.2를 base로 하며, 성능향상 및 운영 고도화, 보안 고도화, ISMS 인증 지원등을 위하여 [OOPS.org](https://oops.org/)에서 개발한 패키지들이 제공이 되며, CentOS 7에서 제공하는 많은 패키지들이 수정이 되었습니다. 이에 대해서는 [***"안녕 리눅스 3 패키지 일람"***](AnNyung3-Package-Catalog.md)을 참조 하십시오.

이 문서에서는 CentOS 7에서 변경된 사항에 대해서만 기술을 합니다. 이 문서에서 다루지 않는 부분은 CentOS 7에서 변경되지 않은 사항들이기 때문에 안녕 리눅스 3을 운영함에 있어 [Redhat Enterprise Linux 7 System Administrator's Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/)를 기본으로 하고, 안녕 리눅스 3의 변경 사항에 대한 부분에 대해 이 문서를 참조 하시기 바랍니다.

## 3. 안녕 리눅스 개발 이력

### 1) 안녕 리눅스 1.0 ~ 1.1

  1. 2003.06.14 1.0 release
  2. 서버 전용으로 사용하기에 점점 비대해지는 Redhat 배포본에 좌절하여, 가볍고 국어 환경에 최적화된 Mini Redhat을 지향
    1. ISO 크기 약 200M, 설치시 400M 정도 사이즈의 작은 크기
    2. fbcon-hanio 패치를 이용하여, console에서 한글 입출력 지원
    3. 문서 및 설정 파일 국문화
    4. 모든 패키지들을 i686 optimize
  3. IBM stack protect patch를 적용하여 Buffer overflow 공격을 원천 방지
  4. Journaling filesystem 도입 (jfs, reiserFS, ext3, xfs 지원)
  5. DevFS 도입
  6. 국내 배포본 중 최초 자동 업데이트 시스템(**autoupdates**) 지원
  7. 서버 전용 배포본
  8. 중소 규모의 서비스 지향

### 2) 안녕 리눅스 1.2 ~ 1.3

  1. Redhat의 그늘에서 탈피 하여 <u>독자적인 배포본 추구</u>
    * 커널의 독자적인 관리
    * glibc를 제외한 거의 모든 패키지들을 안녕 리눅스에서 관리
  2. 패키지 관리 시스템 고도화(Package Systems)
    * autoupdates 를 버리고 처음 부터 새로 개발
    * 자동 업데이트(**pkgsysupdate**) 및 패키지 의존성등을 통합 관리(**pkgadm**)

### 3) 안녕 리눅스 2

  1. 1인 프로젝트 및 非 전업 프로젝트의 한계로 개발 방법 전환
    * Installer 및 설치 image 제공 포기 (CentOS installer에 기생)
    * install image를 제공하지 않는 대신 Yum repository로 제공
    * Yum system 도입 (Packages System 포기)
  2. Enterprise 환경 지향
    * 대형 게임 포털 N사 (DB를 제외한 거의 모든 시스템)
    * 전자상거래 T사 (일부를 제외한 거의 모든 시스템)
    * KLDP 모든 시스템
  3. Kernel과 Glibc는 RHEL/CentOS의 것을 수정하지 않음.
  4. 운영 고도화
  5. 보안 고도화
  6. ISMS 인증 조건 반영
  7. 2012.06.01 RHEL 6u2 / CentOS 6.2 기반으로 안녕 리눅스 2 출시


### 4) 안녕 리눅스 3

  1. 2016.02.29 RHEL 7u2 / CentOS 7.2 기반으로 안녕 리눅스 3 출시
  2. 안녕 리눅스 2의 deploy 계승
