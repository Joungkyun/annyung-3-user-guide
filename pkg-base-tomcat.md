# tomcat

### Description:
Servlet 3.1/JSP 2.3 API를 위한 Apache Servlet/JSP Engine 및 RI

### Changes on AnNyung:
1. http://tomcat.apache.org 의 binary version으로 packaging
2. CentOS의 java 환경이 아니라 AnNyung의 oracle-jre 환경에 맞추어져 있음
 * CentOS의 openjre 환경을 사용하려면, AnNyung의 tomcat을 사용하지 않는다.
 * 이에 대해서는 사용자 가이드의 JAVA 환경에 대한 section을 참
3. tomcat 구동 환경 변수 (/etc/sysconfig/tomcat)
  ```bash
  [root@open ~]$ cat /etc/sysconfig/tomcat
  #
  # tomcat config file
  #
  JAVA_BASE="/usr/java"
  JAVA_HOME="${JAVA_BASE}/default"
  CATALINA_HOME="${JAVA_BASE}/tomcat"

  # Tomcat 구동 account
  TC_RUNUSER=nobody

  # Tomcat 구동시에 설정되는 LANG 환경 변수
  TC_CHARSET=C

  # Instance를 기본 CATALINA_BASE외에 두고 싶을 경우 지정한다.
  TC_INSTANCE_BASE=

  # 여러개의 Instance를 관리할 경우 지정한다. 시작시에 자동 시작을 위해 설정한다.
  TC_INSTANCE_WHITELIST=""

  # MAX FILE OPEN
  TC_MAX_FILE_OPEN=1024

  # STOP TIMEOUT
  TC_STOP_TIMEOUT=5

  TC_AUTHBIND=
  [root@open ~]$
  ```
4. tomcat base dir (CATALINA_HOME)
 * **_/usr/java/tomcat_**

### Sub packages:
