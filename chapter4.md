# Chapter 4. JVM 운영

## 1. 개요



---


!! 경고   
 &lt;<u>6. CentOS JVM 환경으로 rollback</u>&gt; 항목 참고하여 CentOS JVM 환경으로 rollback 하십시오. JVM 환경에 대해서 설계를 다시해야 할 것 같습니다.
 Release 조금 더 늦어질 수도 있을 것 같습니다. T.T
 
 2016년 2월 26일 새벽 2:38 이전에 JVM 환경을 구성하신 분들은 @@@@를 꼭 확인 하십시오.

---

이 문서는 안녕 리눅스의 ***JVM*** 환경에 대해서 기술을 합니다.

안녕 리눅스는 기본으로 ***openJDK 8***과 ***tomcat 8***을 지원합니다. 또한, Oracle JRE/JDK 8 사용을 원하시는 분들을 위하여 Oracle JRE/JDK 8환경 구축에 대한 지원을 합니다. Oracle JDK/JRE에 대해서는 마지막에 별도의 섹션으로 기술 합니다.


## 2. 변경 사항

현재 안녕 리눅스에서 제공하는 JVM 관련 패키지는 다음과 같으며, 이 외의 패키지들은 CentOS의 package를 그대로 사용합니다.

  * [java-1.8.0-openjdk](pkg-base-java-1.8.0-openjdk.md)
  * [tmocat 8](pkg_base-tomcat.md)
  * [javapackages-tools](pkg-base-javapackages-tools.md) (java-1.8.0-oracle package 지원)

안녕 ***JVM*** 환경과 CentOS 환경의 차이는 다음과 같습니다.

1. X dependency가 분리 되었습니다.
  1. ***java-1.8.0-openjdk-headless*** package가 기본 java package 입니다.
    * "`yum install java`" 설치시에 기존 ***java-1.8.0-openjdk*** 가 설치되지 않고 ***java-1.8.0-openjdk-headless*** package가 설치 된다는 의미입니다.
  2. ***java-1.8.0-devel-gui*** package가 새로 추가 되었습니다.
2. ***SDK*** 환경이 기본 ***1.7.0***에서 ***1.8.0***으로 업데이트 되었습니다.
  * "`yum install java-devel`" 설치시에, ***java-1.7.0-openjdk*** package가 설치 되던 것이 이제는 ***java-1.8.0-openjdk*** package들이 설치 됩니다.
3. 안녕 리눅스의 ***openjdk-1.8.0*** package들은 CentOS에서 제공하는 ***openjdk-1.8.0*** rpm package를 풀어서 X dependency 의존성을 분리하고, 기본 패키지(java)를 headless package로 변경한 것입니다. <u>즉, binary와 jar 파일은 CentOS의 package와 동일 합니다.</u>
4. ***tomcat*** version이 8로 업그레이드 되었습니다.
5. 안녕 리눅스에서 지원하는 ***Oracle JRE/JDK*** rpm package에 대한 사항이 ***javapackages-tools*** package에 반영이 되었습니다.



현재 안녕 리눅스의 JVM환경(openjdk 7 + tomcat 8)으로 http://css-validator.kldp.org/ 와 http://validator.kldp.org 의 HTML5 checker(NU) 가 운영 중입니다.


## 3. JVM 설치

1. JRE 설치
  ```bash
  [root@an3 ~]$ yum install java               // X dependency가 필요 없을 경우
  [root@an3 ~]$ yum install java-1.8.0-openjdk // X dependency가 필요 할 경우
  ```

2. JDK 설치
  ```bash
  [root@an3 ~]$ yum install java-devel                   // X dependency가 필요 없을 경우
  [root@an3 ~]$ yum install java-1.8.0-openjdk-devel-gui // X dependency가 필요 할 경우
  ```

3. 설치 후 JAVA 환경 변수
  ```bash
  [root@an3 ~]$ cat test.sh
  #!/bin/bash

  . /usr/share/java-utils/java-functions

  set_jvm
  set_javacmd
  set_jvm_dirs

  echo "JAVACMD     : $JAVACMD"
  echo
  echo "JAVA_HOME   : $JAVA_HOME"
  echo "JAVA_BASE   : $JAVA_BASE"
  echo
  echo "JAVA_LIBDIR : $JAVA_LIBDIR"
  echo "JAVA_LIB    : $JAVA_LIB"
  echo
  echo "JNI_LIBDIR  : $JNI_LIBDIR"
  echo "JNI_LIB     : $JNI_LIB"
  echo
  echo "JVM_ROOT    : $JVM_ROOT"
  echo "JVM_LIBDIR  : $JVM_LIBDIR"
  echo
  echo "CLASSPATH   : $CLASSPATH"
  echo

  #CLASSPATH=$(/usr/bin/build-classpath commons-logging.jar commons-collections.jar servlet-api.jar)
  CLASSPATH+=":$(/usr/bin/build-classpath servlet-api.jar)"
  if [ $? -ne 0 ]; then
      echo "Can't find some jar files"
      exit 1
  fi

  echo "CLASSPATH   : $CLASSPATH"
  echo

  exit 0
  [root@an3 ~]$ bash test.sh
  JAVACMD     : /usr/lib/jvm/jre/bin/java

  JAVA_HOME   : /usr/lib/jvm/jre
  JAVA_BASE   : /usr/lib/jvm

  JAVA_LIBDIR : /usr/share/java
  JAVA_LIB    : /usr/share/java

  JNI_LIBDIR  : /usr/lib/java
  JNI_LIB     : /usr/lib/java

  JVM_ROOT    : /usr/lib/jvm
  JVM_LIBDIR  : /usr/lib/jvm-exports/jre

  CLASSPATH   : /usr/share/java:/usr/lib/java

  CLASSPATH   : /usr/share/java:/usr/lib/java:/usr/share/java/servlet.jar

  [root@an3 ~]$
  ```



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
    * CentOS JVM 환경은 open-jdk 1.7을 제공합니다.
    * oracle-jdk와 openjdk의 차이 - http://stunstun.tistory.com/222 참조
    * 안녕 4에서는 openjdk로 갈아탈 예정입니다. :-)
2. tomcat
  * 배포처에서 EOL 관련 계획에 대한 공지 없음.
  * 2016년 2월 tomcat 6 EOL 공지 (2016.12.31 까지/안녕은 tomcat 8 입니다.)

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
