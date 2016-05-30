# OpenLDAP Master Server 설정


> ***목차***
> 1. package 설치
2. Openldap 초기화
  1. OLC 및 ldap data 초기화 목표
  2. LDAP 설정(olc) 및 LDAP data 초기화
3. account 암호 설정
  1. LDAP 관리자 암호 변경
  2. Account 암호 변경
    1. 관리자가 다른 account의 암호를 변경
    2. 특정 DN의 암호 변경

<br><br>



***참고!***
> 이 문서는 ***RHEL*** >=6, ***CentOS*** >=6, ***AnNyung*** >=2 에서의 설정을 테스트 하였습니다.  
> 이 문서는 <u>***모든 ldap data를 초기화***</u> 시킨 상태에서 설정하는 것을 기준으로 합니다.


##1. package 설치

<br>

***RHEL/CentOS 참고!***

ldap-auth-utils 와 ldap-auth-utils-passwd, genpasswd 패키지는 안녕 리눅스에서만 제공을 합니다. 그러므로 RHEL/CentOS 에서는 안녕 리눅스의 core package repository를 yum에 등록해 주십시오. 아래와 같이 repository를 추가를 하면, 기존의 RHEL/CentOS package를 변경 시키지 않고, 안녕 리눅스에서만 제공하는 패키지를 사용/관리 할 수 있습니다.

***RHEL/CentOS 7*** 에서는 다음과 같이 추가해 주십시오.
```shell
[root@host ~]$ cat &lt;&lt;EOF &gt; /etc/yum.repos.d/AnNyung-core.repos
# AnNyung.repo
#
# LInux AnNyung 3 Yum repository
#

[AN:core]
name=AnNyung 3 Core Repository
mirrorlist=http://annyung.oops.org/mirror.php?release=3&arch=$basearch&repo=core
gpgcheck=1
gpgkey=http://mirror.oops.org/pub/AnNyung/3/RPM-GPG-KEY-AnNyung-3
exclude=php* whois httpd*
[root@host ~]$ yum clean all
```

***RHEL/CentOS 6*** 에서는 다음과 같이 추가해 주십시오.
```shell
[root@host ~]$ cat &lt;&lt;EOF &gt; /etc/yum.repos.d/AnNyung-core.repos
# AnNyung.repo
#
# LInux AnNyung 2 Yum repository
#

[AN:core]
name=AnNyung 2 Core Repository
mirrorlist=http://annyung.oops.org/mirror.php?release=2&arch=$basearch&repo=core
gpgcheck=1
gpgkey=http://mirror.oops.org/pub/AnNyung/2/RPM-GPG-KEY-AnNyung-2
exclude=php* whois httpd*
[root@host ~]$ yum clean all
```

repository 준비가 다 되었다면, ldap-auth-utils 와 ldap-auth-utils-passwd 패키지를 설치 합니다. 이 패키지들을 설치를 하면, genpasswd, openldap-servers, openldap-clients 패키지가 의존성 설정으로 같이 설치가 됩니다.

```bash
[root@an3 ~]$ yum install ldap-auth-utils ldap-auth-utils-passwd
```


##2. Openldap 초기화

<br><br>

> 초기화는 서버의 LDAP 설정및 데이터를 초기화 한 상태에서 진행을 하는 것을 가정 하에 진행을 합니다. 만약 다른 LDAP database를 구동하고 있다면, 이 문서를 참고하는 것을 포기 하십시오!

> 초기화 시에 모든 ldap database를 초기화 시켜 버립니다. 매우 주의 하십시오!!!

<br>

***openldap***은 ***splapd*** daemon을 이용하여 구동이 됩니다. 또한, 2.4.23 버전 부터는 ***slapd.conf*** 대신에 ***OLC(OnLineConfiguration, cn=config 구조)***로 변경이 되었습니다.

물론 기존의 ***slapd.conf***를 migration 해 주는 방법을 제공하고 있기는 하나, 여기서는 그냥 ***OLC***를 이용하는 방법으로 설명을 합니다. ***AnNyung 2*** 에서도 동일하게 ***OLC*** 방식으로 설정을 해야 합니다. (RHEL 6 부터 ***OLC***방식으로 변경이 되었습니다.)



###2.1 OLC 및 ldap data 초기화 목표

ldap 초기화는 다음의 목표를 가지고 진행을 합니다.

1. ***ldap 관리자***로 ***cn=Manager,${BASEND}*** 설정
2. ***ldap 관리자*** 암호 설정
3. UID 0 권한으로 ldap 설정 변경 시에 암호 없이 접근 가능 (-Y EXTERNAL -H ldapi:/// 권한 이용)
4. ldap log를 /var/log/slapd.log 에 남기도록 설정
5. ***ADMIN***, ***People***, ***Group*** OU(Organization Unit) 생성
  * ***Admin*** OU : ***Poeple***, ***Group*** OU를 관리하기 위한 user 및 group account
  * ***People*** OU : ldap 에서 서비스 할 user account
  * ***Group*** OU : ldap 에서 서비스 할 group account
6. host 제한 연동
7. sudo 권한 연동
8. ldap access policy
  1. ldapadmins group의 member는 ***dc=kldp,dc=org*** database에 대한 모든 권한을 가진다.
  2. ldapROusers group의 member는 ***dc=kldp,dc=org*** database에 대한 모든 읽기 권한을 가진다.
  3. 일반 account는 password entry만 제외하고 읽기 권한을 가진다.
  4. 일반 account는 자신의 password entry를 변경할 수 있다.
  5. anonymous account는 접근을 불허 한다.
  6. Ldap server의 system root(UID 0)는 ***cn=manager*** 와 동일한 권한을 가진다.

설치 후, 미리 등록되는 사용자 및 group은 다음과 같습니다.

1. Manager Group
  1. **ldapadmins** : ldap 관리를 할 수 있는 group
  2. **ldapROusers** : ldap 전체 data에 접근할 수 있는 readonly gruop
  3. **ldapmanagers** : ldap을 관리하기 위한 account들의 default group
  4. ldap 관리를 위해서는 사용자를 위의 세 그룹에 권한에 맞게 등록해 주면 자동으로 권한을 획득하게 됩니다.
2. Manager User
  1. **ssoadmin**
    * ldap 관리를 할 수 있는 account
    * ***ldapadmins*** gruop member
    * ***DN***: uid=ssoadmin,ou=Admin,dc=oops,dc=org
  2. **ssomanager**
    * tree 전체의 데이터에 접근 가능한 readonly account
    * ***ldapROusers*** group member
    * 인증 통합시에, 각 client 서버에서 ldap에 연결하기 위한 account
  3. **replica**
    * replication에 사용하기 위한 account (readonly account)
    * ***ldapROusers*** group member
3. System Group
  1. **ldapusers**
    * 생성할 LDAP account들의 default Group


### 2.2 LDAP 설정(olc) 및 LDAP data 초기화

초기화는 ***ldap-auth-utils***의 ***ldap_auth_init*** 명령을 이용합니다.

다시 강조하지만, 이 작업은 기존의 ldap data를 제거하니, 주의 하시기 바랍니다.

먼저, ***ldap_auth_init***를 동작 시키기 위하여, slapd 서비스를 정지 시킨 후 기존의 설정을 제거합니다.

```bash
[root@an3 ~]$ service slapd stop
Redirecting to /bin/systemctl stop  slapd.service
[root@an3 ~]$ rm -rf /etc/openldap/{slapd*.conf,slapd.d}
```

설정 제거 후, ***ldap_auth_init***를 실행 시키고 다음의 과정을 진행 하십시오.

```bash
[root@an3 ~]$ ldap_auth_init
1. ldap 패키지 설치


Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.premi.st
 * epel: ftp.riken.jp
 * extras: ftp.tsukuba.wide.ad.jp
 * updates: mirror.premi.st
Package openldap-servers-2.4.40-9.el7_2.x86_64 already installed and latest version
Package openldap-clients-2.4.40-9.el7_2.x86_64 already installed and latest version
Nothing to do



2. syslog 설정
   . /etc/rsyslog.d/slapd.conf                                    ... Done
     Redirecting to /bin/systemctl restart  rsyslog.service
   . /etc/logrotate.d/openldap                                    ... Done


3. SLAPD 설정 및 구동
   . /etc/logrotate.d/openldap                                    ... Done
     Redirecting to /bin/systemctl restart  slapd.service

4.기본 정보 설정
   BASE DN 입력 : DC=oops,DC=org
   LDAP 관리자 암호 입력   : ****
   LDAP 관리자 암호 재입력 : ****

   결과:
           BASE           => oops
           BASE DN        => DC=oops,DC=org
           ADMIN Password => asdf
           ADMIN Password => {SSHA}pct+aDwN4y4BR3ujyxgWjUBGVqo7fg1/


5. BASE DN 설정
   . /etc/openldap/slapd.d/cn=config/olcDatabase={-1}frontend.ldif ... Done
   . /etc/openldap/slapd.d/cn=config/olcDatabase={0}config.ldif   ... Done
   . /etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif  ... Done
   . /etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif      ... Done


6. 어드민 암호 설정
   SASL/EXTERNAL authentication started
   SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
   SASL SSF: 0
   modifying entry "olcDatabase={0}config,cn=config"
   modifying entry "olcDatabase={2}bdb,cn=config"
     Redirecting to /bin/systemctl restart  slapd.service


7. 기본 트리 구조 초기화
   adding new entry "DC=oops,DC=org"
   adding new entry "ou=Admin,DC=oops,DC=org"
   adding new entry "ou=People,DC=oops,DC=org"
   adding new entry "ou=Group,DC=oops,DC=org"
Done


8. 기본 그룹 생성
   adding new entry "cn=ldapadmins,ou=Admin,DC=oops,DC=org"
   adding new entry "cn=ldapROusers,ou=Admin,DC=oops,DC=org"
   adding new entry "cn=ldapmanagers,ou=Admin,DC=oops,DC=org"
   adding new entry "cn=ldapusers,ou=Group,DC=oops,DC=org"
   adding new entry "uid=ssoadmin,ou=Admin,DC=oops,DC=org"
   adding new entry "uid=ssomanager,ou=Admin,DC=oops,DC=org"
   adding new entry "uid=replica,ou=Admin,DC=oops,DC=org"
Done


9. LDAP 기본 설정
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
   modifying entry "olcDatabase={-1}frontend,cn=config"
Done

[root@an3 ~]$
```

위의 작업을 거치면, ***openldap-servers*** package를 처음 설치를 했을 때 처럼 초기화가 됩니다.


#3 account 암호 설정

##3.1 LDAP 관리자 암호 변경

LDAP 관리자라고 함은, ***slapd***의 관리자를 말합니다.

Section 2의 작업대로 하였을 경우, LDAP의 기본 정보는 다음과 같습니다.

> ***관리자 DN*** : cn=manager,dc=oops,dc=org  
> ***관리자 PW*** : asdf

***BASE DN***에 설정한 account의 경우에는 ***ldappasswd*** 명령을 이용하여 변경을 할 수 있지만, LDAP 관리자의 암호는 ***slapd*** 설정 파일에 포함되어 있기 때문에 ***ldapmodify*** 명령을 이용해야 합니다.

```shell
[root@an3 ~]$ slappasswd -s 'asdf!asdf'
{SSHA}V/udTVfaOUOYEGEyXpVCb6Sy+BHUb244
[root@an3 ~]$
[root@an3 ~]$ cat &lt;&lt;EOF &gt; ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}V/udTVfaOUOYEGEyXpVCb6Sy+BHUb244

dn: olcDatabase={2}bdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}V/udTVfaOUOYEGEyXpVCb6Sy+BHUb244
[root@an3 ~]$
```

##3.2 Account 암호 변경

다음은 ***2.6 LDAP 기본 유저 생성***에서 생성한 account들의 암호를 변경하는 방법입니다. 생성된 account들은 ***ldappasswd*** 명령을 이용하여 변경을 합니다.

일단, 3.2.1 의 방법으로 위에서 생성한 ssoadmin, ssomanager, replica 의 암호를 지정 합니다.

###3.2.1 관리자가 다른 account의 암호를 변경

```bash
[root@an3 ~]$ export BASEDN="dc=oops,dc=org"
[root@an3 ~]$ # Manager 권한으로 ssoadmin 의 암호 변경
[root@an3 ~]$ ldappasswd -H ldapi:/// -x -D "cn=manager,${BASEDN}" -S "uid=ssoadmin,ou=admin,${BASEDN}" -W
New password:          # 지정할 ssoadmin 의 압호 입력
Re-enter new password: # 지정할 ssoadmin의 압호 재입력
Enter LDAP Password:   # ldap 관리자 암호 입력
[root@an3 ~]$
```

### 3.2.2 특정 DN의 암호 변경

***2.7 LDAP Access 정책 설정***의 작업에 의하여, 다른 account의 암호를 변경 하는 것은 LDAP 관리자 와 ssoadmin 유저의 권한(uid=ssoadmin,ou=admin,dc=oops,dc=org)으로 밖에 할 수 없습니다. (ssoadmin 외에 권한을 주려면 account를 생성해서 ldapadmins gruop에 등록해 주면 됩니다.)

그러므로, 여기서는 자신의 계정의 암호를 변경하는 경우가 되겠습니다. 예를 들어 ssoadmin이 자신의 LDAP 암호를 변경하는 경우 입니다.

```bash
[root@an3 ~]$ ldappasswd -H ldapi:/// -D "uid=ssoadmin,ou=admin,dc=kldp,dc=org" -W -S
New password:           # 변경할 암호 입력
Re-enter new password:  # 변경할 암호 재입력
Enter LDAP Password:    # 기존의 ssoadmin 암호 입력
[root@an3 ~]$
```

위의 ***ldappasswd*** 명령에서 ***-S*** 옵션을 주지 않으면, 임의의 8자리 암호를 생성해서 변경을 합니다. 또한, ***-A*** 옵션을 주면 현재의 암호(old password)를 match 시켜야 암호를 변경할 수 있습니다만, 별로 의미가 없어 보여 여기서는 사용하지 않았습니다.

변경한 암호로 로그인이 되는지 확인해 봅니다.

```bash
[root@an3 ~]$ ldapsearch -H ldap:/// -D "uid=ssoadmin,ou=admin,dc=oops,dc=org" -W
Enter LDAP Password: # 변경 전 암호 입력
ldap_bind: Invalid credentials (49)
[root@an3 ~]$ ldapsearch -H ldapi:/// -D "uid=ssoadmin,ou=admin,dc=kldp,dc=org" -W
Enter LDAP Password: # 변경한 암호 입력
# extended LDIF
#
# LDAPv3
# base <> (default) with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 32 No such object

# numResponses: 1
[root@an3 ~]$
```

여기까지 작업을 하면, Master 서버의 설정은 SSL 설정을 마치고는 대략 다 되었습니다.

