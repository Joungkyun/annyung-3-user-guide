# OpenLDAP Master Server 설정


> ***목차***
> 1. package 설치
2. Openldap 초기화
  1. OLC 및 ldap data 초기화 목표
  2. LDAP 설정(olc) 및 LDAP data 초기화
  3. LDAP client 설정
  4. 초기화 확인
3. account 암호 설정
  1. LDAP 관리자 암호 변경
  2. Account 암호 변경


<br><br>



***참고!***
> 이 문서는 ***RHEL*** >=6, ***CentOS*** >=6, ***AnNyung*** >=2 에서의 설정을 테스트 하였습니다.  
> 이 문서는 <u>***모든 ldap data를 초기화***</u> 시킨 상태에서 설정하는 것을 기준으로 합니다.


##1. package 설치

<br>

***RHEL/CentOS 참고!***

이 문서에서 설명하는 LDAP 관리도구인 ldap-auth-utils 와 ldap-auth-utils-passwd, genpasswd 패키지는 안녕 리눅스에서만 제공을 합니다. 그러므로 RHEL/CentOS 에서는 안녕 리눅스의 core package repository를 yum에 등록해 주십시오. 아래와 같이 repository를 추가를 하면, 기존의 RHEL/CentOS package를 변경 시키지 않고, 안녕 리눅스에서만 제공하는 패키지를 사용/관리 할 수 있습니다.

***RHEL/CentOS 7*** 에서는 다음과 같이 추가해 주십시오.
```shell
[root@host ~]$ cat <<EOF > /etc/yum.repos.d/AnNyung-core.repos
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
[root@host ~]$ cat <<EOF > /etc/yum.repos.d/AnNyung-core.repos
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
  1. ldapadmins group의 member는 ***dc=oops,dc=org*** database에 대한 모든 권한을 가진다.
  2. ldapROusers group의 member는 ***dc=oops,dc=org*** database에 대한 모든 읽기 권한을 가진다.
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

### 2.3 LDAP client 설정

초기화를 한 LDAP database 를 관리하기 위한 client 설정은 ***/etc/openldap/ldap.conf*** 와 ***/etc/openldap/ldap-auth-utils.conf*** 에서 합니다.

먼저 ***/etc/openldap/ldap.conf***에 ***ldap_auth_init***에서 설정한 BASE DN과 LDAP 서버 URI를 설정 합니다.

```shell
[root@an3 ~]$ cat <<EOF >> /etc/openldap/ldap.conf
URI ldapi:///
BASE DC=oops,DC=org
EOF
[root@an3 ~]$
```

다음, ***/etc/openldap/ldap-auth-utils.conf***를 설정 합니다. ***ldap-auth-utils.conf*** 에서는 ***ldap_auth_init***로 초기화를 한 경우에는 별로 건드릴 것이 없습니다만, 몇몇 튜닝이 가능한 옵션에 대해서만 여기서 설명을 합니다. 자세한 사항은 "***man ldap-auth-utils.conf***" 명령으로 man page를 참조 하십시오.

***ldap-auth-utils.conf***는 ldap account를 관리하기 위한 기본 값을 지정 합니다.

***ldap-auth-utils*** 패키지에서 관리할 최소 UID/GID, 기본 Group ID, 기본 shell, home directory prefix, 암호 알고리즘, 암호 만기, 암호 복잡도 등의 설정을 가지고 있습니다.

일단, ***ldap_auth_init***를 이용하여 초기화를 한 경우, 일단 고려를 한 것은 암호 알고리즘에 대해서 고민을 해야 합니다. ***ldap-auth-utils.conf***에서 암호 알고리즘은 다음의 옵션값으로 지정 합니다.

```shell
[root@an3 ~]$ cat /etc/openldap/ldap-auth-utils.conf | grep "^PASSWD_MECH"
PASSWD_MECH                  = md5
[root@an3 ~]$
```

RHEL/CentOS 6 이나 AnNyung 2 부터는는 ***PAM***에서 기본으로 ***sha512**를 지원합니다. 하지만 ***RHEL/CentOS 5***등의 오래된 배포본들은 기본값을 ***md5*** 입니다. 그러므로 암호 매커니즘의 호환성을 위하여 ***ldap-auth-utils.conf***의 암호화 기본 알고리즘은 ***md5*** 로 되어 있습니다.

만약, 인증 통합을 할 서버들이 모두 최소한 ***RHEL/CentOS 6*** 또는 ***AnNyung 2*** 이상이거나, 또는 ***PAM***에서 ***sha512***를 지원하도록 설정이 되어 있으면, ***PASSWD_MECH***의 값을 ***sha512***로 변경을 하시면 password strenth가 더욱 좋아지게 됩니다.

***RHEL/CentOS 5***에서 sha512를 사용하기를 원한다면 다음의 명령을 이용할 수 있습니다.

```shell
[root@RHEL5 ~]$ authconfig --passalgo=sha512 --update
```

이미 ***md5***를 사용하고 있는 시스템에서 변경을 한다면 기존의 암호들을 갱신해 줘야 합니다. 이에 대해서는 http://www.cyberciti.biz/faq/rhel-centos-fedora-linux-upgrading-password-hashing/ 문서를 참고 하십시오.


또한, ***ISMS*** 심사에 대비하여, 암호 만기 설정이 90일로 되어 있으니, 이 기간이 너무 짧다고 생각이 되면 ***PASS_MAX_DAYS*** 값을 늘려 주십시오.

암호 복잡도 설정은 ***PASS_MINLEN*** 와 ***PASS_CLASSES*** 에서 할 수 있습니다.

>주의!  
>이 설정은 ***ldap account***를 위한 설정입니다. 즉, ***ldap_adduser*** 명령을 이용할 경우 적용이 되는 것들로, system의 ***adduser***와는 별개 입니다.

```shell
[root@an3 ~]$ cat /etc/openldap/ldap-auth-utils.conf
  .. 상략..
#
# Password configuration
#

# Specify the encryption method.
# support: md5, sha256, sha512
# default: md5
#
# As of RHEL 5, if there is a host that does not support sha512,
# please specify the md5 for compatibility. From RHEL 6, it
# supports sha512.
PASSWD_MECH                  = md5

#
# Pasword expire configuration
#
# If the value is not specified, the value that is set to
# /etc/login.defs will be used.
#
#   PASS_MAX_DAYS   Maximum number of days a password may be used.
#   PASS_MIN_DAYS   Minimum number of days allowed between password changes.
#   PASS_WARN_AGE   Number of days warning given before a password expires.
#
PASS_MAX_DAYS                = 90
PASS_MIN_DAYS                = 0
PASS_WARN_AGE                = 7

#
# Strength of password
#
# minimum length of password
PASS_MINLEN                  = 9

# Password complexity configuration [ 0 - 4 ]
#
# vSetting of the complexity of the password. Number of the character classes.
# Character class is upper case and lower case letters, numbers and special
# characters.
PASS_CLASSES                 = 3
[root@an3 ~]#
```


##2.4 초기화 확인

***ldap_auth_init***을 이용하여 초기화를 한 후에, 다음과 같이 ldap 관리 account와 gruop, 그리고 ldap account의 default group이 잘 등록이 되었는지 확인을 합니다.

```shell
[root@an3 ~]# ldap account 확인
[root@an3 ~]$ ldapsearch -Y EXTERNAL  "(objectClass=posixAccount)" dn
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
# extended LDIF
#
# LDAPv3
# base <DC=kldp, DC=org> (default) with scope subtree
# filter: (objectClass=posixAccount)
# requesting: dn
#

# ssoadmin, Admin, oops.org
dn: uid=ssoadmin,ou=Admin,dc=oops,dc=org

# ssomanager, Admin, kldp.org
dn: uid=ssomanager,ou=Admin,dc=oops,dc=org

# replica, Admin, kldp.org
dn: uid=replica,ou=Admin,dc=oops,dc=org

# search result
search: 2
result: 0 Success

# numResponses: 4
# numEntries: 3
[root@an3 ~]$
[root@an3 ~]$ # ldap group 확인
[root@an3 ~]$ ldapsearch -Y EXTERNAL  "(objectClass=posixGroup)" dn
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
# extended LDIF
#
# LDAPv3
# base <DC=kldp, DC=org> (default) with scope subtree
# filter: (objectClass=posixGroup)
# requesting: dn
#

# ldapadmins, Admin, kldp.org
dn: cn=ldapadmins,ou=Admin,dc=oops,dc=org

# ldapROusers, Admin, kldp.org
dn: cn=ldapROusers,ou=Admin,dc=oops,dc=org

# ldapmanagers, Admin, kldp.org
dn: cn=ldapmanagers,ou=Admin,dc=oops,dc=org

# ldapusers, Group, kldp.org
dn: cn=ldapusers,ou=Group,dc=oops,dc=org

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 4
[root@an3 ~]$
```


#3 account 암호 설정

##3.1 LDAP 관리자 암호 변경

LDAP 관리자라고 함은, ***slapd***의 관리자를 말합니다.

Section 2의 작업대로 하였을 경우, LDAP의 기본 정보는 다음과 같습니다.

> ***관리자 DN*** : cn=manager,dc=oops,dc=org  
> ***관리자 PW*** : asdf

***BASE DN***에 설정한 account의 경우에는 ***ldappasswd*** 명령을 이용하여 변경을 할 수 있지만, LDAP 관리자의 암호는 ***slapd*** 설정 파일에 포함되어 있기 때문에 ***ldapmodify*** 명령을 이용해야 합니다.

***slappasswd*** 명령을 이용하여 암호를 encrypt 한 후에, ldif 형식을 이용하여 업데이트 할 수 있습니다.

```shell
[root@an3 ~]$ export CHGPASSWD=$(slappasswd -s 'asdf!asdf')
{SSHA}V/udTVfaOUOYEGEyXpVCb6Sy+BHUb244
[root@an3 ~]$
[root@an3 ~]$ cat <<EOF > ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: ${CHGPASSWD}

dn: olcDatabase={2}bdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: ${CHGPASSWD}
[root@an3 ~]$ unset CHGPASSWD
```

##3.2 Account 암호 변경

***ldap_auth_init***으로 LDAP database를 초기화를 하면, 기본적으로 ***ssoadmin***, ***ssomanager***, ***replica*** account가 암호 없이 생성이 됩니다. 특히 ***ssoadmin***의 경우에는 ***BASE DN***의 데이터를 변경할 수 있는 권한을 가지고 있고, ***ssomanager*** 와 ***replica*** account는 인증 통합시에 사용하는 권한이기 때문에 데이터베이스 전반에 대한 read 권한을 가지고 있습니다.

그러므로, ***ldap_auth_init***으로 데이터베이스를 초기화 했을 경우, 꼭 이 3 계정의 암호를 지정해 주어야 합니다. 

암호 변경은 ***ldap_passwd*** 명령을 이용합니다. ***openldap***의 ***ldappasswd***와 비슷하니 주의 하십시오. ***ldap_passwd***는 root 권한으로 실행을 해야 하며, <u>보안 상 ***localhost***의 ldap database만 관리할 수 있습니다.</u>

***ldap_passwd***를 사용할 경우, account는 ***"ACCOUNT@DOMAIN"***의 형식을 사용합니다. 즉, ***BASE DN***이 ***DC=oops,DC=org*** 라면, ***DOMAIN***은 ***oops.org***가 됩니다.

ssoadmin, ssomanager, replica account의 OU는 People이 아니라 Admin 이기 때문에 ***-u*** 옵션으로 OU를 변경해 줘야 합니다. -u 옵션을 주지 않으면 기본으로 People OU를 사용하게 됩니다.

```shell
[root@an3 ~]$ # ssoadmin account 암호 변경
[root@an3 ~]$ ldap_passwd -u Admin ssoadmin@kldp.org
New password     : ***********
Re-New password  : ***********

Your Informations:

    * Account: ssoadmin@kldp.org
    * RDN : uid=ssoadmin,ou=Admin,dc=kldp,dc=org
    * Host: ldapi:///
    * Privilieges: -Y EXTERNAL
    * Commnad: /usr/bin/ldapmodify -H "ldapi:///" -Y EXTERNAL
    * Hash: {CRYPT}$6$nTeAanL57QGjvsdv$fd6YrbOqI4hFTJEZxm5GIuu8HUGMfcNz6Tx0AmaEsC78TJ0SBh/lEYqCI1KP.95H3TeTBsA1wX7nBnprwv0Hz0

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "uid=ssoadmin,ou=Admin,dc=kldp,dc=org"

0
Done
[root@an3 ~]$
[root@an3 ~]$ # ssomanager account 암호 변경
[root@an3 ~]$ ldap_passwd -u Admin ssomanager@kldp.org
New password     : ***********
Re-New password  : ***********

Your Informations:

    * Account: ssoadmin@kldp.org
    * RDN : uid=ssomanager,ou=Admin,dc=kldp,dc=org
    * Host: ldapi:///
    * Privilieges: -Y EXTERNAL
    * Commnad: /usr/bin/ldapmodify -H "ldapi:///" -Y EXTERNAL
    * Hash: {{CRYPT}$6$htk01t9cUA5aFM/a$9H4.kig058cRESS6MGdjn8armHHP6IAQO9Qykr6iroW9laqugz.bIOPNzBUgk8N4H01QkeklEwQg05FBzSrfz/

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "uid=ssoadmin,ou=Admin,dc=kldp,dc=org"

0
Done
[root@an3 ~]$
[root@an3 ~]$ # replica account 암호 변경
[root@an3 ~]$ ldap_passwd -u Admin replica@kldp.org
New password     : ***********
Re-New password  : ***********

Your Informations:

    * Account: ssoadmin@kldp.org
    * RDN : uid=replica,ou=Admin,dc=kldp,dc=org
    * Host: ldapi:///
    * Privilieges: -Y EXTERNAL
    * Commnad: /usr/bin/ldapmodify -H "ldapi:///" -Y EXTERNAL
    * Hash: {{CRYPT}$6$Qo.Btjze5bY/pXGl$zUltslxPOandgoPrXb3OkESk5R7pgqAmU.Q8gGsbYxGPsx9mMnSEWVH/kdHuW4Gn7lIIBx5zr260LRoxPSMWL.

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "uid=ssoadmin,ou=Admin,dc=kldp,dc=org"

0
Done
[root@an3 ~]$
```






