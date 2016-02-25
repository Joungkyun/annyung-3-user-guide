# oracle-jre

### Description:
The Java Platform Standard Edition Runtime Environment (JRE) contains
everything necessary to run applets and applications designed for the
Java platform. This includes the Java virtual machine, plus the Java
platform classes and supporting files.

The JRE is freely redistributable, per the terms of the included license.

### Features:
1. Oracle JRE의 업데이트는 년 3회 security update 버전을 따라 업데이트 됩니다.
2. Oracle JRE binary version을 rpm packaging 하였습니다.
3. oracle-jre profile
  ```bash
  [root@an3 ~]$ cat /etc/profile.d/jre.sh
  # JRE Profile

  JAVA_BASE="/usr/java"
  JAVA_HOME="/usr/java/default"
  PATH="${PATH}:${JAVA_HOME}/bin"

  export JAVA_BASE JAVA_HOME PATH
  [root@an3 ~]$
  ```
4. 안녕 리눅스 3의 JAVA 환경에 대해서는 사용자 매뉴얼을 참조 하십시오.

### Reference:
* http://www.oracle.com/technetwork/java/javase/downloads/index.html

### Dependencies:
* None

### Sub Packages:
* None

### Releated Packages:
* None