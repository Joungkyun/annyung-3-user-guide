# Openldap 인증 통합

현재 작성 중입니다. ___\*\\^^/\*___

openldap을 이용한 인증 통합은 openldap을 multi-master replaction 으로 구성을 하는 방법으로 기술을 합니다.

##1. Master server 설정

##1.1 package 설치

```bash
[root@an3 ~]$ yum install openldap-servers openldap-clients genpasswd
```

##1.2 Openldap 초기화

***openldap***은 ***splapd*** daemon을 이용하여 구동이 됩니다. 또한, 2.4.23 버전 부터는 ***slapd.conf*** 대신에 ***OLC(OnLineConfiguration, cn=config 구조)***로 변경이 되었습니다.

물론 기존의 ***slapd.conf***를 migration 해 주는 방법을 제공하고 있기는 하나, 여기서는 그냥 ***OLC***를 이용하는 방법으로 설명을 합니다. ***AnNyung 2*** 에서도 동일하게 ***OLC*** 방식으로 설정을 해야 합니다. (RHEL 6 부터 ***OLC***방식으로 변경이 되었습니다.)


###1.2.1 OLC 초기화

일단, 처음 ***openldap-servers*** package를 설치 하신 경우 또는, openldap 설정을 한번도 안한 상태라면, 이 단계는 뛰어 넘어도 상관이 없습니다.

이 단계는, 기존의 openldap 설정을 모두 날려 버리고, 설치 시의 상태로 복원 시키는 것이니, 이 작업을 할지에 대해서는 신중하게 판단을 하십시오.

```bash
[root@an3 ~]$ # slapd.conf와 slapd.d 디렉토리를 제거 합니다.
[root@an3 ~]$ rm -f /etc/openldap/slapd.conf
[root@an3 ~]$ rm -rf /etc/openldap/slapd.d
[root@an3 ~]$ # slapd.d 를 생성하고 초기화 합니다.
[root@an3 ~]$ mkdir -p /etc/openldap/slapd.d
[root@an3 ~]$ slaptest -f /usr/share/openldap-servers/slapd.conf.obsolete \
                     -F /etc/openldap/slapd.d
[root@an3 ~]$ # slapd.d 의 디렉토리 파일의 권한에서 group/extra 권한을 모두 제거 합니다.
[root@an3 ~]$ chown -R ldap:ldap /etc/openldap/slapd.d
[root@an3 ~]$ chmod -R 000 /etc/openldap/slapd.d
[root@an3 ~]$ chmod -R u+rwX /etc/openldap/slapd.d
[root@an3 ~]$ #  기존의 openldap data를 모두 초기화 시킵니다.
[root@an3 ~]$ rm -rf /var/lib/ldap/*
[root@an3 ~]$ cp -af /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@an3 ~]$ chown ldap.ldap /var/lib/ldap/DB_CONFIG
```

위의 작업을 거치면, ***openldap-servers*** package를 처음 설치를 했을 때 처럼 초기화가 됩니다.

###1.2.2 syslog 설정

***slapd*** daemon의 log를 남기기 위한 설정을 합니다.

```bash
[root@an3 ~]$ mkdir -p /var/log/slapd
[root@an3 ~]$ chown -R ldap:ldap /var/log/slapd
[root@an3 ~]$ cat > /etc/syslog.d/slapd <<EOF
local4.*  /var/log/spamd/slapd.log
EOF
[root@an3 ~]$ service rsyslog restart
시스템 로거 종료 중:                                       [  OK  ]
시스템 로거 시작 중:                                       [  OK  ]
[root@an3 ~]$
```

다음 logrotate 설정을 합니다.

```bash
[root@an3 ~]$ cat /etc/logrotate.d/openldap <<EOF
var/log/slapd/*.log {
    copytruncate
    rotate 4
    monthly
    notifempty
    missingok
    compress
    create 0644 ldap ldap
}
EOF
[root@an3 ~]$
```

###1.2.3 slapd 설정

먼저, ***LDAP***에서 사용할 ***Base DN***을 정합니다. ***Base DN***은 mysql의 database 라고 생각을 하시면 됩니다.

보통은 사용하시는 도메인을 많이 사용합니다. 사용하시는 domain이 ***oops.org***이라면

>***DC=oops,DC=org***

위와 같이 지정하면 되며, 여기서는 이 설정을 ***Base DN***의 예제로 사용할 것입니다.

다음, 위에서 정한 ***Base DN***을 설정 합니다.

```bash
[root@an3 ~] # dc=oops,dc=org 부분은 설정한 것으로 지정해야 합니다.
[root@an3 ~]$ perl -pi -e 's/dc=my-domain,dc=com/dc=oops,dc=org/g' \
                          /etc/openldap/slapd.d/cn\=config/olcDatabase*
```

다음, ***slapd***를 시작 시킵니다. 이는 ***slapd***가 ***OLC(OnLineConfiguration)*** 이기 때문에 설정이 초기화 상태에서 시작을 먼저 시켜 주는 것입니다.

```bash
[root@an3 ~]$ service slapd start
slapd 설정 파일 확인 중:                                   [주의]
5708bfb7 ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif"
5708bfb7 ldif_read_file: checksum error on "/etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif"
config file testing succeeded
slapd (을)를 시작 중:                                      [  OK  ]
[root@an3 ~]$ # 부팅시에 slapd가 시작되도록 해 줍니다.
[root@an3 ~]$ systemctl enable slapd
```

위에서 slapd를 시작할 때 checksum error가 발생하는 이유는, ***openldap***이 ***OCL***로 변경된 이후 설정 파일은 ***ladpadd***, 또는 ***ldapmodify*** 등으로 설정을 변경해야 하는데, 위에서 ***DN*** 설정을 수동으로 변경을 했기 때문에 발생하는 것입니다. (물론 운영에 문제가 있지는 않습니다.)

이 에러 메시지를 없애기 위해서 다음과 처리를 합니다.

아래 명령은 cat부터 EOF열까지를 복사해서 실행 시키시면 됩니다.

```bash
[root@an3 ~]$ cat <<EOF | ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcDatabase={-1}frontend,cn=config
changetype: modify
replace: olcReadOnly
olcReadOnly: FALSE

dn: olcDatabase={0}config,cn=config
changetype: modify
replace: olcReadOnly
olcReadOnly: FALSE

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcReadOnly
olcReadOnly: FALSE

dn: olcDatabase={2}bdb,cn=config
changetype: modify
replace: olcReadOnly
olcReadOnly: FALSE
EOF
[root@an3 ~]$
```

***ldapmodify*** 명령으로 변경된 설정 파일들의 ***checksum***을 갱신합니다. 물론 위의 설정값들은 이 작업을 하는 목적이 ***checksum*** 갱신이기 때문에, 동일한 값으로 변경을 하는 것입니다. (이 작업은 1회만 하면 됩니다. 다음 부터는 직접 설정 파일을 수정하지 않기 때문입니다.)

다시 ***slapd***를 재시작 해서 checksum 에러가 발생하는지 확인 합니다.

```bash
[root@an3 ~]$ service slapd restart
slapd 설정 파일 확인 중:                                   [주의]
config file testing succeeded
slapd (을)를 시작 중:                                      [  OK  ]
[root@an3 ~]$
```


###1.2.4 Admin password 설정

***slappasswd*** 명령을 이용하여 사용할 암호의 hash 문자열을 생성합니다. (여기서는 "***asdf!2345***"의 hash 문자열로 진행을 합니다.)

```bash
[root@an3 ~]$ slappasswd
New password:
Re-enter new password:
{SSHA}PYxLS1BKElJx3ER3zwCfHX6nhu5a1H2l
[root@an3 ~]$
```

만약, system의 passwd entry에 있는 문자열을 그대로 사용하고 싶으시다면, 

> ***{CRYPT}$1$ggRKVU3b$TZduI8fIrxZ9LpJ9NqAJZ1***

와 같이 사용을 해도 됩니다. 위의 hash 문자열은 sha512방식의 crypt 암호화된 hash로, md5 방식의 암호화 입니다. 즉, /etc/shadow의 암호화 문자열 앞에 ***{CRYPT}*** 만 prefix로 붙여 주시면 됩니다.

그리고, 다음의 명령으로 Admin 암호를 설정 합니다.

```bash
[root@an3 ~]$ cat <<EOF | ldapmodify -Y EXTERNAL -H ldapi:///
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}PYxLS1BKElJx3ER3zwCfHX6nhu5a1H2l

dn: olcDatabase={2}bdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}PYxLS1BKElJx3ER3zwCfHX6nhu5a1H2l
EOF
[root@an3 ~]$
```

###1.2.5 LDAP Tree 생성

인증에 필요한 Ldap database tree(***OU- Organiztion Unit***)를 생성합니다. 여기서는 Admin, Users, Groups database를 생성할 예정이며, 다음의 용도로 사용합니다.

> ***Aadmin*** : Ldap 관리를 위한 user와 group entryp 저장  
> ***Users***  : system account entry 저장  
> ***Groups*** : system group entry 저장

LDAP tree에서 ***DN***을 mysql의 database에 비유했다면, ***OU***는 table 정도라고 생각하면 됩니다. 위의 Admin, Users, Groups 는 ***OU***에 해당이 됩니다. 이 ***OU***안에 passwd, group entry가 들어가게 되며, mysql가 다른 점은 ***OU*** 밑에 또 ***OU***가 있을 수 있다는 점입니다. file system과 비유를 하면 ***Directory***로 생각하는 것이 더 맞을지도 모르겠습니다. 이 부분에 대해서 정확하게 알고 싶다면, LDAP 문서들을 읽어 보시는 것을 권장 합니다. 

***DN*** 설정은 *설정하신 **DN***으로 수정을 하셔야 합니다.

```bash
[root@an3 ~]$ cat > addtree.ldif <<EOF
dn: dc=oops,dc=org
dc: oops
o: OOPS LDAP
objectclass: dcObject
objectclass: organization
objectclass: top

dn: ou=Admin,dc=oops,dc=org
ou: Users
objectclass: organizationalUnit

dn: ou=Users,dc=oops,dc=org
ou: Users
objectclass: organizationalUnit

dn: ou=Groups,dc=oops,dc=org
ou: Groups
objectclass: organizationalUnit
EOF
[root@an3 ~]$ ldapadd -a -c -H ldapi:/// -D "cn=Manager,dc=oops,dc=org" -W -f addtree.ldif
Enter LDAP Password: # input admin password
adding new entry "dc=oops,dc=org"
adding new entry "ou=Admin,dc=oops,dc=org"
adding new entry "ou=Users,dc=oops,dc=org"
adding new entry "ou=Groups,dc=oops,dc=org"
[root@an3 ~]$
```

###1.2.6 LDAP 기본 유저 생성

Ldap을 관리하기 위한 기본 user/group entry를 생성 하며, 다음의 권한을 가집니다. 이 account, group들은 아래의 ACL에 의해서 자동으로 권한을 가지게 됩니다.

1. Manager Group
  1. **ldapadmins** : ldap 관리를 할 수 있는 group
  2. **ldapROusers** : ldap 전체 data에 접근할 수 있는 readonly gruop
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
    * * ***ldapROusers*** group member
3. System Group
  1. **ldapusers**

```bash
[root@an3 ~]$ export BASEDN="dc=oops,dc=org"
[root@an3 ~]$ cat adduser.ldif <<EOF
# extended LDIF
#
# LDAPv3
# base <${BASEDN}> with scope subtree
# filter: (cn=*)
# requesting: ALL
#

# ldapadmins, Admin
dn: cn=ldapadmins,ou=Admin,${BASEDN}
objectClass: posixGroup
objectClass: top
cn: ldapadmins
description: LDAP Management group
gidNumber: 9999
memberUid: ssoadmin

# ldapROusers, Admin
dn: cn=ldapROusers,ou=Admin,${BASEDN}
objectClass: posixGroup
objectClass: top
cn: ldapROusers
description: LDAP Read only group
gidNumber: 9998
memberUid: replica
memberUid: ssomanager

# ldapusers, Groups
dn: cn=ldapusers,ou=Groups,${BASEDN}
objectClass: posixGroup
objectClass: top
cn: ldapusers
description: LDAP account groups
gidNumber: 10000

# ssoadmin, Admin
dn: uid=ssoadmin,ou=Admin,${BASEDN}
objectClass: posixAccount
objectClass: top
objectClass: inetOrgPerson
objectClass: shadowAccount
gidNumber: 9997
givenName: SSO
sn: Admin
displayName: SSO Admin
uid: ssoadmin
homeDirectory: /
gecos: SSO Aadmin
loginShell: /sbin/nologin
shadowFlag: 0
shadowMin: 0
shadowMax: 99999
shadowWarning: 0
shadowInactive: 99999
shadowLastChange: 12011
shadowExpire: 99999
cn: SSO Admin
uidNumber: 9999

dn: uid=ssomanager,ou=Admin,${BASEDN}
objectClass: posixAccount
objectClass: top
objectClass: inetOrgPerson
objectClass: shadowAccount
gidNumber: 9997
givenName: SSO
sn: Manager
displayName: SSO Manager
uid: ssomanager
homeDirectory: /
gecos: SSO Manager
loginShell: /sbin/nologin
shadowFlag: 0
shadowMin: 0
shadowMax: 99999
shadowWarning: 0
shadowInactive: 99999
shadowLastChange: 12011
shadowExpire: 99999
cn: SSO manager
uidNumber: 9998

dn: uid=replica,ou=Admin,${BASEDN}
objectClass: posixAccount
objectClass: top
objectClass: inetOrgPerson
objectClass: shadowAccount
gidNumber: 9997
givenName: Replica
sn: User
displayName: Replica User
uid: replica
homeDirectory: /
gecos: Replica User
loginShell: /sbin/nologin
shadowFlag: 0
shadowMin: 0
shadowMax: 99999
shadowWarning: 0
shadowInactive: 99999
shadowLastChange: 12011
shadowExpire: 99999
cn: Replica User
uidNumber: 9997
EOF
[root@an3 ~]$ ldapadd -a -c -H ldapi:/// -D "cn=Manager,${BASEDN}" -W -f adduser.ldip
Enter LDAP Password: # input admin password
[root@an3 ~]$
```

아직 해당 account에 대한 암호가 설정이 되지 않은 상태 이므로, 생성한 account를 사용할 수 있는 단계는 아닙니다. 암호는 ***replication*** 설정 후에, 설정할 예정입니다.

###1.2.7 LDAP Access 정책 설정

이 부분은 각 account/group의 정책 및 LDAP의 기본 보안을 수립하게 됩니다.

정책의 모티브는 다음과 같습니다.

1. ldapadmins group의 member는 ***dc=kldp,dc=org*** database에 대한 모든 권한을 가진다.
2. ldapROusers group의 member는 ***dc=kldp,dc=org*** database에 대한 모든 읽기 권한을 가진다.
3. 일반 account는 password entry만 제외하고 읽기 권한을 가진다.
4. 일반 account는 자신의 password entry를 변경할 수 있다.
5. anonymous account는 접근을 불허 한다.

```bash
[root@an3 ~]$ export BASEDN="dc=oops,dc=org"
[root@an3 ~]$cat <<EOF | ldapmodify -Y EXTERNAL -H ldapi:/// | sed 's/^/   /g' | grep -v "^[ ]*$"
dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcAccess
olcAccess: to dn.base="" by * read
olcAccess: to dn.base="cn=subschema" by * read
olcAccess: to dn.subtree="ou=Users,${BASEDN}" attrs=userPassword,shadowLastChange by set="[cn=ldapadmins,ou=Admin,${BASEDN}]/memberUid & user/uid" manage by set="[cn=ldapROusers,ou=Admin,${BASEDN}]/memberUid & user/uid" write by self =wx by anonymous auth
olcAccess: to dn.subtree="ou=Groups,${BASEDN}" by users read by anonymous auth
olcAccess: to dn.subtree="ou=Users,${BASEDN}" by users read by anonymous auth
olcAccess: to * by set="[cn=ldapadmins,ou=Admin,${BASEDN}]/memberUid & user/uid" manage by set="[cn=ldapROusers,ou=Admin,${BASEDN}]/memberUid & user/uid" read by anonymous auth
EOF
[root@an3 ~]$
```
