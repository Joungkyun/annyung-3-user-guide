# Chapter 4. JVM 운영

## 1. 개요

안녕 리눅스의 JVM 환경은 ***oracle JRE 8***과 ***tomcat 8***을 지원합니다.

안녕 리눅스에서 제공하는 ***oracle JRE 8***과 ***tomcat 8***은 oracle과 tomcat 에서 제공하는 ***binary package***들을 풀어서 경로만 재구성 한 다음 ***re-packaing*** 한 것이기 때문에, 다운로드 버전과 차이가 없습니다.

## 2. 권고 사항

현재 안녕 리눅스에서 제공하는 JVM 관련 패키지는 다음과 같으며, 이 외의 패키지들은 CentOS의 package를 그대로 사용합니다.

  * oracle-jre 1.8
  * tomcat 8
  * javapackages-tool (X/GUI 관련 의존성 제거)

다음의 조건에 해당한다면, ***CentOS JVM 환경***을 이용하십시오. ***CentOS JVM 환경***을 이용하는 방법은 <u>***"6. CentOS JVM 환경으로 rollback"*** 섹션을 참고</u> 하십시오.

1. X 또는 GUI 관련 리소스가 필요한 경우  
  안녕 리눅스의 JVM 환경은 X 의존성이 없고, 간단한 web service정도를 하기 위해 구성된 환경 입니다.
2. 안녕/CentOS에서 제공하는 marven package를 사용하여 marven을 구성하려고 할 경우.
3. 그냥 안녕의 JVM 환경은 못 믿겠다..

일단, JVM 환경은 워낙 다양한 경우가 많아서 일일이 대응을 하기가 힘이 듭니다. 그러므로, 무언가 문제가 있다고 판단이 되면, <u>CentOS JVM 환경으로 rollback</u> 하여 테스트 해 보시기 바랍니다.

현재 안녕 리눅스의 JVM환경으로 http://css-validator.kldp.org/ 와 http://validator.kldp.org 의 HTML5 checker(NU) 가 운영 중입니다.

## 3. Oracle JDK

Oracle JDK의 경우, 재배포 라이센스가 제한되어 있기 때문에 <u>binary package를 제공하지 않습니다.</u> 다만, 안녕 리눅스 3의 ***srpms repository***에 보시면 ***oracle-jdk*** source rpm을 제공 합니다. ***oracle-jdk*** source rpm은 재배포 제한 때문에 jdk binary를 포함하고 있지 않기 때문에 매우 사이즈가 작습니다. (binary package를 빌드시 직접 다운로드 받음)

그러므로 다음의 작업으로 oracle-jdk package를 구하실 수 있습니다.

```bash
[root@an3 ~]$ yum install rpm-build rpm-devel yum-utils
[root@an3 ~]$ mkdir -p /root/rpmbuild/{BUILD,BUILDROOT,SOURCES,SPECS,SRPMS}
[root@an3 ~]$ mkdir -p /root/rpmbuild/RPMS/{noarch,x86_64}
```

위와 같이 rpm package를 생성할 수 있도록 준비를 합니다. 이 작업은 1회만 하면 됩니다. 패키지는 일반 유저로도 만들 수 있지만, 여기서는 일단 root 권한으로 하는 것을 가정합니다.

```bash
[root@an3 ~]$ yumdownload --source oracle-jdk
[root@an3 ~]$ rpmbuild --rebuild oracle-jdk-1.8.0-71.an3.src.rpm
[root@an3 ~]$ cd /root/rpmbuild/RPMS/x86_64
[root@an3 ~]$ ls
oracle-jdk-1.8.0-71.el7.x86_64.rpm
```

생성된 package를 별도의 yum repository로 만드셔서 관리 하시면 됩니다.


## 4. JVM/tomcat Life time

안녕 리눅스에서 제공하는 JVM(oracle-jre)와 tomcat package는 해당 패키지 배포처의 binary package를 사용하는 것이기 때문에, 이들 package의 life time은 각 패키지의 life time 까지만 제공 합니다.

1. JAVA SE 8
  * http://www.oracle.com/technetwork/java/eol-135779.html
  * End of Public Updates Notification : TBD
  * End of Public Updates : 2017.09
    * long term 유지보수를 원하신다면, open-jdk 사용을 고려해 보시기 바랍니다.
    * CentOS JVM 환경은 open-jdk를 제공합니다.
    * http://stunstun.tistory.com/222 참조
    * 안녕 4에서는 openjdk로 갈아탈 예정입니다. :-)
2. tomcat
  * 배포처에서 EOL 관련 계획에 대한 공지 없음.
  * 2016년 2월 tomcat 6 EOL 공지 (2016.12.31 까지)

## 5. 안녕 리눅스 oracle-jre 업데이트 주기

안녕 리눅스의 oracle-jre 업데이트는 oracle에서 제공하는 년 3회의 ***Critical Patch Update*** release에 대응 합니다.

Oracle의 2016년 ***Critical Patch Update*** 예정은 다음과 같습니다.

  * http://www.oracle.com/technetwork/topics/security/alerts-086861.html
  * 2016.04.19
  * 2016.07.19
  * 2016.10.18
  * 2017.01.17


## 6. CentOS JVM 환경으로 rollback

이 섹션은 안녕 리눅스의 JVM환경을 CentOS JVM 환경으로 rollback 하는 것을 설명 합니다.

1. 이미 JVM 환경을 구성을 한 적이 있다면, 설치한 package들을 rollback 하시기 바랍니다.
   일단, 안녕에서 제공하는 JVM package는 다음과 같습니다.
   * oracle-jre
   * tomcat
   * javapackages-tool
2. 안녕 Yum repository [AN:base]와 [AN:addon]에서 관련 패키지를 차단합니다.  
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
1. CentOS Yum repository 설정에서 JVM 관련 제한을 풀어줍니다.  
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
