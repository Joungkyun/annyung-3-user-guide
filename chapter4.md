# Chapter 4. JVM 운영

## 1. 개요

안녕 리눅스의 JVM 환경은 oracle JRE 8과 tomcat 8을 지원합니다.

안녕 리눅스에서 제공하는 oracle JRE 8과 tomcat 8은 oracle과 tomcat 에서 제공하는 binary package들을 풀어서 경로만 재구성 한 다음 re-packaing 한 것이기 때문에, 다운로드 버전과 차이가 없습니다.

## 2. 권고 사항

안녕 리눅스의 JVM 환경은 X 의존성이 없고, 간단한 web service정도를 하기 위해 구성된 환경 입니다. 복잡한 구성이 필요할 경우, 안녕의 package를 사용하지 말고 직접 구성하시기 바랍니다.

또는 CentOS의 JVM 환경을 사용하기를 원한다면, "CentOS JVM 환경으로 rollback" 섹션을 참조 하십시오.

특히, CentOS에서 제공하는 marven package를 사용할 경우, 꼭 CentOS JVM 환경으로 전환 하기 바랍니다.

## 3. JVM/tomcat Life time

안녕 리눅스에서 제공하는 JVM(oracle-jre)와 tomcat package는 해당 패키지 배포처의 binary package를 사용하는 것이기 때문에, 이들 package의 life time은 각 패키지의 life time 까지만 제공 합니다.

## 4. CentOS JVM 환경으로 rollback

이 섹션은 안녕 리눅스의 JVM환경을 CentOS JVM 환경으로 rollback 하는 것을 설명 합니다.

1. 안녕 Yum repository [AN:base]와 [AN:addon]에서 관련 패키지를 차단합니다.  
  아래의 예와 같이 설정 같이 ***"exclude=XXXX"*** 라인을 추가해 주십시오.
  ```bash
  [root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.repo

  ** 상략 **

  [AN:base]
  name=AnNyung $annyungver Base Repository
  mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
  #baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/base/$basearch
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
  exclude=tomcat* oracle-jre *java*

    ** 중략 **
    
  [AN:addon]
  name=AnNyung $annyung-ver addon Repository
  mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=addon
  #baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/addon/$basearch
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
  exclude=tomcat* oracle-jre *java*

    ** 하략 **

  [root@an3 ~]$
  ```
2. CentOS Yum repository 설정에서 JVM 관련 제한을 풀어줍니다.  
   ***/etc/yum.repos.d/CentOS-Base.repo*** 에서 아래의 예와 같이 ***모든*** ***exclude=XXX*** 라인에서 java와 tomcat을 제거 합니다.  
   아래의 예는 [update] repository만 예를 들었지만 파일에 있는 모든 repository에 대해서 수정해 주셔야 합니다.

   ```bash
   [updates]
   name=CentOS-$releasever - Updates
   mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   #baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   #exclude=php* pear* httpd* mysql* mariadb* *java* *tomcat*
   exclude=php* pear* httpd* mysql* mariadb*
   ```
