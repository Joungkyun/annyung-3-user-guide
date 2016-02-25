# Chapter 4. JVM 운영

> 목차
1. 개요
2. 변경 사항
3. JVM 설치
4. TOMCAT
5. Oralce JDK
6. CentOS JVM 환경으로 rollback

## 1. 개요

---


!! 경고   

2016년 2월 26일 새벽 2:38 이전에 JVM 환경을 구성하신 분들은 "***&lt;6.2 2016년 2월 26일 이전 환경 migration&gt;***"를 꼭 확인 하십시오.

---

이 문서는 안녕 리눅스의 ***JVM*** 환경에 대해서 기술을 합니다.

안녕 리눅스는 기본으로 ***openJDK 8***과 ***tomcat 8***을 지원합니다. 또한, Oracle JRE/JDK 8 사용을 원하시는 분들을 위하여 Oracle JRE/JDK 8환경 구축에 대한 지원을 합니다. Oracle JDK/JRE에 대해서는 ***&lt;5. Oracle JDK&gt;*** 항목을 참고 하십시오.

JDK 6 사용시, CentOS에서 ***java-1.6.0-openjdk*** package를 지원합니다만, 권장 하지 않습니다. ***openjdk***는 1.7.0 부터 Oracle JDK와 대등한 수준의 API를 지원합니다. (대등한 성능을 의미하는 것이 아니라 API 대응을 의미하는 것입니다.) 그러므로 <u>JDK 6 환경은 Oracle JDK를 받아서 직접 구성하십시오.</u>


## 2. 변경 사항

현재 안녕 리눅스에서 제공하는 JVM 관련 패키지는 다음과 같으며, 이 외의 패키지들은 CentOS의 package를 그대로 사용합니다.

  * [java-1.8.0-openjdk](pkg-base-java-1.8.0-openjdk.md) (X 의존성 분리)
  * [tmocat 8](pkg_base-tomcat.md)
  * [javapackages-tools](pkg-base-javapackages-tools.md) (java-1.8.0-oracle package 지원)

안녕 ***JVM*** 환경과 CentOS 환경의 차이는 다음과 같습니다.

1. X dependency가 분리 되었습니다.
  1. ***java-1.8.0-openjdk-headless*** package가 기본 java package 입니다.
    * "`yum install java`" 설치시에 기존 ***java-1.8.0-openjdk*** 가 설치되지 않고 ***java-1.8.0-openjdk-headless*** package가 설치 된다는 의미입니다.
  2. ***java-1.8.0-devel-gui*** package가 새로 추가 되었습니다.
2. ***SDK*** 환경이 기본 ***1.7.0***에서 ***1.8.0***으로 업데이트 되었습니다.
  * CentOS에서 "`yum install java-devel`" 설치 시에, ***java-1.7.0-openjdk*** package가 설치 되던 것이 안녕 리눅스에서는 ***java-1.8.0-openjdk*** package들이 설치 됩니다.
3. 안녕 리눅스의 ***openjdk-1.8.0*** package들은 CentOS에서 제공하는 ***openjdk-1.8.0*** rpm package를 풀어서 X dependency 의존성을 분리하고, 기본 패키지(java)를 headless package로 변경한 것입니다. <u>즉, binary와 jar 파일은 CentOS의 package와 동일 합니다.</u>
4. ***tomcat*** version이 8로 업그레이드 되었습니다.
5. 안녕 리눅스에서 지원하는 ***Oracle JRE/JDK*** rpm package에 대한 사항이 ***javapackages-tools*** package에 반영이 되었습니다.
6. ***java-1.7.0*** package와 ***javav-1.6.0*** package는 변경 사항이 없습니다. (X 의존성이 그대로 있습니다.)

안녕 리눅스의 ***java-1.8.0-openjdk***을 사용을 원하지 않는다면 ***/etc/yum.repos.d/AnNyung.repo***의 ***AN:base*** repository에서 ***java-1.8.0***을 exclude 시키십시오.

```bash
[root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.repo
  .. 상략 ..
[AN:base]
name=AnNyung $annyungver Base Repository
mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
#baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/base/$basearch
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
exclude=java-1.8.0*
  .. 하략 ..
[root@an3 ~]
```

현재 안녕 리눅스의 JVM환경(openjdk 7 + tomcat 8)으로 http://css-validator.kldp.org/ 와 http://validator.kldp.org 의 HTML5 checker(NU) 가 운영 중입니다.


## 3. JVM 설치

1. JRE 설치
  ```bash
  [root@an3 ~]$ yum install java               // X dependency가 필요 없을 경우
  [root@an3 ~]$ yum install java-1.8.0-openjdk // X dependency가 필요 할 경우
  [root@an3 ~]$ yum install java-1.7.0-openjdk-headless // No X dependency
  [root@an3 ~]$ yum install java-1.7.0-openjdk          // X dependency
  ```

2. JDK 설치
  ```bash
  [root@an3 ~]$ yum install java-devel                   // X dependency가 필요 없을 경우
  [root@an3 ~]$ yum install java-1.8.0-openjdk-devel-gui // X dependency가 필요 할 경우
  [root@an3 ~]$ yum install java-1.7.0-openjdk-devel     // 무조건 X dependency가 있음
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
  
##4. TOMCAT

안녕 리눅스의 TOMCAT은 버전만 8로 업그레이드 되었으며, 환경 구성은 CentOS와 동일 합니다.

1. tomcat 설치
  ```bash
  [root@an3 ~]$ yum install tomcat
  ```
2. 설정 파일 : */etc/tomcat*
  ```bash
  [root@an3 ~]$ ls -l /etc/tomcat
  drwxrwxr-x 3 root   tomcat     22  2월 26 05:02 Catalina/
  -rw-rw-r-- 1 tomcat tomcat  12374  2월 25 17:24 catalina.policy
  -rw-rw-r-- 1 tomcat tomcat   7106  2월 25 17:24 catalina.properties
  -rw-rw-r-- 1 tomcat tomcat   1577  2월 25 17:24 context.xml
  -rw-rw-r-- 1 tomcat tomcat   3387  2월 25 17:24 logging.properties
  -rw-rw-r-- 1 tomcat tomcat   6458  2월 25 17:24 server.xml
  -rw-rw---- 1 tomcat tomcat   2212  2월 25 17:24 tomcat-users.xml
  -rw-rw-r-- 1 tomcat tomcat    574  2월 25 17:25 tomcat.conf
  -rw-rw-r-- 1 tomcat tomcat 168822  2월 25 17:24 web.xml
  [root@an3 ~]$
  ```
3. 로그 파일 : */var/log/tomcat*
  ```bash
  [root@an3 ~]$ ls -l /var/log/tomcat/
  -rw-r--r-- 1 tomcat tomcat 23916  2월 25 19:01 catalina.2016-02-25.log
  -rw-r--r-- 1 tomcat tomcat     0  2월 25 17:56 host-manager.2016-02-25.log
  -rw-r--r-- 1 tomcat tomcat     0  2월 25 17:56 localhost.2016-02-25.log
  -rw-r--r-- 1 tomcat tomcat 14181  2월 25 19:00 localhost_access.2016-02-25.log
  -rw-r--r-- 1 tomcat tomcat     0  2월 25 17:56 manager.2016-02-25.log
  [root@an3 ~]$
  ```
4. _CATALINA_HOME_ : */usr/share/tomcat*
  ```bash
  [root@an3 ~]$ l /usr/share/tomcat/
  drwxr-xr-x 2 root root   73  2월 26 05:02 bin
  lrwxrwxrwx 1 root tomcat 11  2월 26 05:02 conf -> /etc/tomcat
  lrwxrwxrwx 1 root tomcat 22  2월 26 05:02 lib -> /usr/share/java/tomcat
  lrwxrwxrwx 1 root tomcat 15  2월 26 05:02 logs -> /var/log/tomcat
  lrwxrwxrwx 1 root tomcat 22  2월 26 05:02 temp -> /var/cache/tomcat/temp
  lrwxrwxrwx 1 root tomcat 23  2월 26 05:02 webapps -> /var/lib/tomcat/webapps
  lrwxrwxrwx 1 root tomcat 22  2월 26 05:02 work -> /var/cache/tomcat/work
  [root@an3 ~]$
  ```
5. 기본 DocumentRoot : */var/lib/tomcat/webapps* 또는 */usr/share/tomcat/webapps*
6. tomcat 구동
```bash
[root@an3 ~]$ service tomcat [eanble|disable]              // 부팅시 구동 여부
[root@an3 ~]$ service tomcat [start|stop|restart|status]   // tomcat 구동
```

***Tomcat 7***을 사용하기를 원한다면 ***/etc/yum.repos.d/AnNyung.repo***의 ***AN:base*** repository에 tomcat을 exclude 시키십시오.

```bash
[root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.repo
  .. 상략 ..
[AN:base]
name=AnNyung $annyungver Base Repository
mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
#baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/base/$basearch
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
exclude=tomcat*
  .. 하략 ..
[root@an3 ~]
```


## 5. Oracle JDK

***oracle-jre***나 ***oracle-jdk***가 설치된 환경은 2016년 2월 26일 이전에 제공하던 환경입니다. 안녕 리눅스 3릴리즈 직전에 안녕 리눅스의 JVM환경이 변경이 되었으니, 이전에 환경을 구성했다면 초기화 하시기 바랍니다.

```bash
[root@an3 ~]$ yum remove oracle-jre oracle-jdk tomcak
```


### 1. package 빌드

기본적으로, ***Oracle JDK***의 경우 재배포를 허용하지 않으므로 직접 다운로드 받아서 설치를 해야 합니다.

안녕 리눅스에서는 ***Oracle JDK***를 위하여 java-1.8.0-oracle source rpm을 제공합니다. 재배포를 허용하지 않기 때문에 source rpm으로 rebuild 시에 다운로드를 받아서 rpm을 제작 하며, 제작된 rpm들은 CentOS에서 제공하는 JVM환경과 호환이 됩니다.

또한, 제작된 RPM은 Oracle에서 제공하는 JDK 8 rpm을 받아서 <u>file path및 Provides 정보</u>를 ***CentOS JVM*** 환경과 동일하게 re-packaing 한 것이므로, <u>binary level에서는 Oracle에서 제공하는 ***binary(rpm)***과 동일합니다.</u>


다음의 작업으로 oracle-jdk package를 생성할 수 있으며, build 시간은 대략 2~5분 정도 소요됩니다.

```bash
[root@an3 ~]$ yum install rpm-build rpm-devel yum-utils
[root@an3 ~]$ mkdir -p /root/rpmbuild/{BUILD,BUILDROOT,SOURCES,SPECS,SRPMS}
[root@an3 ~]$ mkdir -p /root/rpmbuild/RPMS/{noarch,x86_64}
```

위와 같이 rpm package를 생성할 수 있도록 준비를 합니다. 이 작업은 1회만 하면 됩니다. 패키지는 일반 유저로도 만들 수 있지만, 여기서는 일단 root 권한으로 하는 것을 가정합니다. (버전은 달라질 수 있습니다.)

```bash
[root@an3 ~]$ yumdownloader --source java-1.8.0-oracle
[root@an3 ~]$ rpmbuild --rebuild oracle-jdk-1.8.0.73-1.an3.src.rpm
[root@an3 ~]$ cd /root/rpmbuild/RPMS/x86_64
[root@an3 x86_64]$ ls -l
-rw-r--r-- 1 root root 18295560  2월 25 19:51 java-1.8.0-oracle-1.8.0.73-1.an3.x86_64.rpm
-rw-r--r-- 1 root root 93091660  2월 25 19:51 java-1.8.0-oracle-devel-1.8.0.73-1.an3.x86_64.rpm
-rw-r--r-- 1 root root  1231420  2월 25 19:51 java-1.8.0-oracle-devel-gui-1.8.0.73-1.an3.x86_64.rpm
-rw-r--r-- 1 root root 35614032  2월 25 19:51 java-1.8.0-oracle-headless-1.8.0.73-1.an3.x86_64.rpm
-rw-r--r-- 1 root root 24767532  2월 25 19:51 java-1.8.0-oracle-src-1.8.0.73-1.an3.x86_64.rpm
[root@an3 x864_64]
```

### 2. Yum repository 생성하기

생성된 package를 별도의 yum repository로 만들어 관리 합니다.

```bash
[root@an3 ~] yum insatll createrepo
[root@an3 ~] mkdir -p oracle-jdk/x86_64
[root@an3 ~] mv /root/rpmbuild/RPMS/x86_64/java-1.8.0-*.rpm oracle-jdk/x86_64/
[root@an3 ~] cd oracle-jdk
[root@an3 oracle-jdk]$ createrepo --workers 5 --changelog-limit 10 ./
[root@an3 oracle-jdk]$ cat OracleJDK.repo
[Oracle JDK]
name=AnNyung $annyungver Oracle JDK Repository
baseurl=http://domain.com/oracle-jdk/$basearch
;gpgcheck=1
;gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
[root@an3 oracle-jdk]$
```

이렇게 하면 Oracle JDK에 대한 yum repository를 생성하게 됩니다. 이 디렉토리를 Web에서 접근이 가능하돌고 설정을 한 후, OracleJDK.repo를 /etc/yum.repos.d에 복사를 합니다.


### 3. 설치

```bash
[root@an3 ~]$ yum install java       // install java-1.8.0-oracle-headless (JRE)
[root@an3 ~]$ yum install java-devel // install java-1.8.0-orcale-devel (JDK)
```

X libaray나 audio, video API를 사용하기 위해서는 다음의 명령으로 설치 하셔야 합니다.

```bash
[root@an3 ~]$ yum install java-1.8.0-oracle           // install JRE
[root@an3 ~]$ yum install java-1.8.0-oracle-devel-gui // install JDK
```

***java-1.7.0-openjdk***와 ***java-1.8.0-openjdk*** package와 같이 설치 할 수있습니다.



### 4. Oracle JVM EOL(End Of Lifetime)

안녕 리눅스에서 제공하는 ***Oracle JVM(java-1.8.0-oracle)***은 해당 패키지 배포처의 binary package를 사용하는 것이기 때문에, 이들 package의 life time은 ***Oracle JDK***의 life time 까지만 제공 합니다.

1. JAVA SE 8
  * http://www.oracle.com/technetwork/java/eol-135779.html
  * End of Public Updates Notification : TBD
  * End of Public Updates : 2017.09
    * long term 유지보수를 원하신다면, open-jdk 사용을 고려해 보시기 바랍니다.
    * oracle-jdk와 openjdk의 차이 - http://stunstun.tistory.com/222 참조
    * 안녕 4에서는 oracle-jdk를 제공하지 않을 예정입니다.


### 5. 안녕 리눅스 oracle-jre 업데이트 주기

안녕 리눅스의 oracle-jre 업데이트는 oracle에서 제공하는 년 3회의 ***Critical Patch Update*** release에 대응 합니다.

Oracle의 2016년 ***Critical Patch Update*** 예정은 다음과 같습니다.

  * http://www.oracle.com/technetwork/topics/security/alerts-086861.html
  * 2016.04.19
  * 2016.07.19
  * 2016.10.18
  * 2017.01.17


## 6. CentOS JVM 환경으로 rollback

이 섹션은 안녕 리눅스의 JVM환경을 사용하지 않는 방법을 설명합니다. 또한, 안녕 3 릴리즈 전(정확히는  2016.02.26 전)에 안녕의 JVM 환경을 설치했을 경우의 rollback에 대한 설명을 기술 합니다.

### 1. 안녕 리눅스 JVM 환경 사용 하지 않기


1. 설치되어 있는 패키지를 제거 합니다. (설치 전이라면 2번으로 넘어 갑니다.)
  ```bash
  [root@an3 ~]$ yum remove "tomcat*" "java-1.8.0-oracle*"
  ```

2. ***[AN:base]*** repository에서 ***tomcat***과 ***java-1.8.0-openjdk***를 배포하지 않도록 설정 합니다.

  ```bash
  [root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.repo
    .. 상략 ..
  [AN:base]
  name=AnNyung $annyungver Base Repository
  mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
  #baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/base/$basearch
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
  exclude=java-1.8.0-openjdk* tomcat*
    .. 하략 ..
  [root@an3 ~]
  ```
3. Yum cache를 reset 후 재설치를 합니다.
  ```bash
  [root@an3 ~]$ yum clean all
  [root@an3 ~]$ yum install java tomcat
  ```
  
### 2. 2016년 2월 26일 이전 환경 migration

이 섹션은 2016년 2월 26일 이전, oracle-jre package와 tomcat package를 제공하던 환경을 안녕 리눅스 3 공식적인 환경으로 migration 하는 방법을 기술 합니다.

1. 기존의 package를 제거 합니다.
  ```bash
  [root@an3 ~]$ yum remove "tomcat*" oracle-jre oracle-jdk
  ```
2. ***/etc/yum.repos.d/AnNyung.repo*** 파일에서 exclude= 항목에 아래와 같이 ***tomcat***, ***oracle-jre***, ***oracle-jdk***, ___&midast;java&midast;___ 이 있으면 제거 합니다.
  ```bash
  [root@an3 ~]$ cat /etc/yum.repos.d/AnNyung.repo

  ** 상략 **

  [AN:base]
  name=AnNyung $annyungver Base Repository
  mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=base
  #baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/base/$basearch
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
  #exclude=tomcat* oracle-jre *java*

    ** 중략 **
    
  [AN:addon]
  name=AnNyung $annyung-ver addon Repository
  mirrorlist=http://annyung.oops.org/mirror.php?release=$annyungver&arch=$basearch&repo=addon
  #baseurl=http://mirror.oops.org/pub/AnNyung/$annyungver/addon/$basearch
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-AnNyung-$annyungver
  #exclude=tomcat* oracle-jre *java*

    ** 하략 **

  [root@an3 ~]$
  ```

3. CentOS Yum repository 설정에서 JVM 관련 제한을 풀어줍니다.  
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
