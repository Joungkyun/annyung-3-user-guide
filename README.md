안녕 리눅스 3 사용자 가이드
=======

Copyright &copy; 2016 [JoungKyun.Kim](https://oops.org/) all rights reserved.

## 1. 이력

* 2016.02.29 초판 발행
* 설치에 대해서는 [안녕 리눅스 3 설치 문서](https://joungkyun.gitbooks.io/annyung3-installation-guide/content/)를 참조 하십시오.
* 모든 문서가 작성 완료 되지는 않았으며, 계속 작성 중입니다.
* 2016.03.11 ***[Chapter 6. Time Server 운영](chapter6.md)*** 문서 업데이트
* 2016.04.05 ***[Chapter 2.3.2 NIS 인증 통합](chapter2-3-auth-integrate-nis.md)*** 문서 업데이트
* 2016.06.05 ***[Chapter 2.3.1 Openldap 인증 통합](chapter2-3-auth-integrate-openldap.md)*** 문서 업데이트

## 2. 개요

이 문서는 안녕 리눅스 3를 운영 관리하기 위한 정보를 전달 합니다.

기본적으로 안녕 리눅스는 CentOS 7.2를 base로 하며, 성능향상 및 운영 고도화, 보안 고도화, ISMS 인증 지원등을 위하여 [OOPS.org](https://oops.org/)에서 개발한 패키지들이 제공이 되며, CentOS 7에서 제공하는 많은 패키지들이 수정이 되었습니다. 이에 대해서는 [***"안녕 리눅스 3 패키지 일람"***](AnNyung3-Package-Catalog.md)을 참조 하십시오.

이 문서에서는 CentOS 7에서 변경된 사항에 대해서만 기술을 합니다. 이 문서에서 다루지 않는 부분은 CentOS 7에서 변경되지 않은 사항들이기 때문에 안녕 리눅스 3을 운영함에 있어 [Redhat Enterprise Linux 7 System Administrator's Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/)를 기본으로 하고, 안녕 리눅스 3의 변경 사항에 대한 부분에 대해 이 문서를 참조 하시기 바랍니다.


## 3. 안녕 리눅스란?
 
안녕 리눅스는 RHEL(Redhat Enterprise Linux) 계열의 배포본으로, RHEL/CentOS와 최대한 호환되도록 만들어진 배포본 입니다. 안녕 리눅스 1.x에서는 독자적인 배포본을 지향하였고, enterprise 환경 보다는 중소 규모의 독립적인 웹서버에 적합하도록 개발이 되었으나, 안녕 리눅스 2 부터는 RHEL/CentOS와의 최대한 호환을 유지하는 정책을 이어 enterprise 환경을 지향 합니다.

RHEL/CentOS와의 최대한의 호환을 유지한다는 의미는, RHEL/CentOS를 운영함과 거의 동일함을 의미합니다. RHEL/CentOS의 정책을 유지하면서, 보안성과 운영의 활용도를 높일 수 있는 기능과 최신 패키지가 추가 되었습니다.

안녕 리눅스 2의 경우, Apache / PHP / JVM 환경을 제외하고는 RHEL/CentOS와 동일한 정책으로 운영이 가능하며, 안녕 리눅스 3의 경우에는 Apache / PHP 환경을 제외하고 RHEL /CentOS 와 동일한 정책으로 운영이 가능 합니다.

즉, CentOS/RHEL 사용자라면, 안녕 리눅스로 전환하는데 드는 비용이 거의 0에 가깝다는 것을 의미합니다.


## 4. 안녕 리눅스 개발 이력

### 4.1 안녕 리눅스 1.0 ~ 1.1

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

### 4.2 안녕 리눅스 1.2 ~ 1.3

  1. Redhat의 그늘에서 탈피 하여 <u>독자적인 배포본 추구</u>
    * 커널의 독자적인 관리
    * glibc를 제외한 거의 모든 패키지들을 안녕 리눅스에서 관리
  2. 패키지 관리 시스템 고도화(Package Systems)
    * autoupdates 를 버리고 처음 부터 새로 개발
    * 자동 업데이트(**pkgsysupdate**) 및 패키지 의존성등을 통합 관리(**pkgadm**)

### 4.3 안녕 리눅스 2

  1. 1인 프로젝트 및 非 전업 프로젝트의 한계로 개발 방법 전환
    * Installer 및 설치 image 제공 포기 (CentOS installer에 기생)
    * install image를 제공하지 않는 대신 Yum repository로 제공
    * Yum system 도입 (Packages System 포기)
  2. Enterprise 환경 지향
    * 대형 게임 포털 N사 (DB를 제외한 거의 모든 시스템)
    * 전자상거래 T사 (일부를 제외한 거의 모든 시스템)
    * KLDP 모든 시스템
  3. 서버 전용 배포본
    1. 최초 설치 시 1.1G 정도 사이즈의 적은 용량 (설치 후 package를 어떻게 설치하느냐에 따라 달라지지만, 서버 운영이 목적이라면 최대 4G를 넘을 일은 없을 것으로 판단 합니다.)
    2. X 관련 패키지 및 의존성을 최대한 배제
    3. jfbterm을 이용하여, console에서 한글 입출력 지원(현재 입력은 버그가 있어 안됨)
    4. 최초 설치시 SSH daemon외에 서비스 daemon 실행 안됨.
  4. Kernel과 Glibc는 RHEL/CentOS의 것을 수정하지 않음.
  5. 운영 고도화
  6. 보안 고도화
  7. ISMS 인증 조건 반영
  8. 2012.06.01 RHEL 6u2 / CentOS 6.2 기반으로 안녕 리눅스 2 출시
   * Apache / PHP / JVM 환경 호환 되지 않음. 사용자 가이드 참고


### 4.4 안녕 리눅스 3

  1. 2016.02.29 RHEL 7u2 / CentOS 7.2 기반으로 안녕 리눅스 3 출시
   * Apache / PHP 환경 호환 되지 않음. 사용자 가이드 참고
   * JVM 환경은 호환 됨. (안녕 2와의 차이)
  2. 안녕 리눅스 2의 deploy 계승
