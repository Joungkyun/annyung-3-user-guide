# OpenLDAP Master Server 설정

> 참고!  
> 3 항목에서 2 까지의 작업을 수행하는 script 예제를 제시하고 있습니다. 2 까지는 어떠한 작접이 필요한지 참고 하시고, 3 의 script sample을 이용하여 설정 하십시오.

##1 package 설치

```bash
[root@an3 ~]$ yum install openldap-servers openldap-clients genpasswd
```

***genpasswd*** package는 안녕 리눅스에서만 제공 합니다. RHEL이나 CentOS에서는 안녕 리눅스의 repository 에 있는 getpasswd rpm package를 수동으로 받아서 설치 하시면 사용이 가능 합니다. RHEL/CentOS 6과 7용이 제공 됩니다. RHEL 6/7(CentOS 6/7) 외의 버전이나 다른 배포본에서는 srpm을 받아서 빌드 하여 사용하십시오.

##2 Openldap 초기화

***openldap***은 ***splapd*** daemon을 이용하여 구동이 됩니다. 또한, 2.4.23 버전 부터는 ***slapd.conf*** 대신에 ***OLC(OnLineConfiguration, cn=config 구조)***로 변경이 되었습니다.

물론 기존의 ***slapd.conf***를 migration 해 주는 방법을 제공하고 있기는 하나, 여기서는 그냥 ***OLC***를 이용하는 방법으로 설명을 합니다. ***AnNyung 2*** 에서도 동일하게 ***OLC*** 방식으로 설정을 해야 합니다. (RHEL 6 부터 ***OLC***방식으로 변경이 되었습니다.)


###2.1 OLC 초기화

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

###2.2 syslog 설정

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

###2.3 slapd 설정

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
[root@an3 ~]$ perl -pi -e 's/#SLAPD_/SLAPD_/g' /etc/sysconfig/ldap
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

아래 명령은 cat 라인부터 마지막 EOF 까지를 copy 한 후, paste 하고 실행하면 됩니다.

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


###2.4 Admin password 설정

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

변경한 LDAP 관리자의 DN은 "***cn=manager,{BASE_DN}***"이며, 여기서는 ***cn=manager,dc=oops,dc=org***"가 되며, 다음의 방법으로 확인을 합니다.

```bash
[root@an3 ~]$ # "-W" 옵션으로 암호 입력을 하지 않을 경우 에러 발생
[root@an3 ~]$ ldapsearch -x -D "cn=manager,dc=oops,dc=org"
ldap_bind: Server is unwilling to perform (53)
        additional info: unauthenticated bind (DN with no password) disallowed
[root@an3 ~]$ # 잘못된 암호를 입력했을 경우
[root@an3 ~]$ ldapsearch -x -D "cn=manager,dc=oops,dc=org" -W
Enter LDAP Password:  # 틀린 암호 입력
ldap_bind: Invalid credentials (49)
[root@an3 ~]$ # 암호 입력이 제대로 되었을 경우
[root@an3 ~]$ ldapsearch -x -D "cn=manager,dc=kldp,dc=org" -W
Enter LDAP Password:
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


###2.5 LDAP Tree 생성

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

###2.6 LDAP 기본 유저/그룹 생성

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
    * ***ldapROusers*** group member
3. System Group
  1. **ldapusers**
    * 생성할 LDAP account들의 기본 Group

아래의 명령으로 user/group을 생성 합니다.

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

아직 해당 account에 대한 암호가 설정이 되지 않은 상태 이며, 아래에서 passwd 설정 하는 방법을 따로 설명 합니다.

다음은, Groups OU에 ldapusers group이 잘 생성이 되었는지 확인하여 봅니다.

```bash
[root@an3 ~]$ # cn=manager,dc=oops,dc=org 권한으로 ou=Groups,dc=oops,dc=org 의 entry 탐색
[root@an3 ~]$ ldapsearch -x -D "cn=manager,dc=oops,dc=org" -W -b "ou=Groups,dc=oops,dc=org"
Enter LDAP Password: # LDAP 관리자 암호(여기서의 예는 "asdf!2345") 입력
# extended LDIF
#
# LDAPv3
# base <ou=Groups,dc=kldp,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Groups, oops.org
dn: ou=Groups,dc=oops,dc=org
ou: Groups
objectClass: organizationalUnit

# ldapusers, Groups, oops.org
dn: cn=ldapusers,ou=Groups,dc=oops,dc=org
objectClass: posixGroup
objectClass: top
cn: ldapusers
description: LDAP account groups
gidNumber: 10000

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
[root@an3 ~]$
```

###2.7 LDAP Access 정책 설정

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

#3 account 암호 설정

##3.1 LDAP 관리자 암호 변경

LDAP 관리자라고 함은, ***slapd***의 관리자를 말합니다. ***2.4 Admin password 설정***에서 설정한 암호 "***asdf!2345***"는 이 LDAP 관리자의 암호 입니다.

위의 작업대로 하였을 경우, LDAP의 기본 정보는 다음과 같습니다.

> ***관리자 DN*** : cn=manager,dc=oops,dc=org  
> ***관리자 PW*** : asdf!2345

***BASE DN***에 설정한 account의 경우에는 ***ldappasswd*** 명령을 이용하여 변경을 할 수 있지만, LDAP 관리자의 암호는 ***slapd*** 설정 파일에 포함되어 있기 때문에 ***ldapmodify*** 명령을 이용해야 합니다.

***2.4 Admin password 설정*** 항목을 참고 하셔서 변경을 하시면 됩니다.

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


##4. LDAP database init script

여기 까지 작업을 하시면, 인증 통합을 위한 Master server의 기본 설정이 완료된 상태 입니다. 상당히 복잡한 작업인데, 이 항목에서는 위의 작업들을 간단히 할 수 있는 script 예제를 제공 합니다.

이 스크립트를 이용하면, 1.2.7 까지의 작업을 수행하게 됩니다.

```bash
[root@an3 ~]$ cat /etc/openldap/ldap-data-init.sh
#!/bin/bash

passwd_prompt() {
    local prompt="$* : "
    prompt_value=""
    while IFS= read -p "$prompt" -r -s -n 1 char
    do
        if [[ $char == $'\0' ]]
        then
            break
        fi
        prompt='*'
        prompt_value+="$char"
    done
    echo
}

success() {
    echo -en "\\033[1;32m"
    [ -n "$1" ] && echo -n "$*" || echo "OK"
    echo -en "\\033[0;39m"
}

failure() {
    echo -en "\\033[1;31m"
    [ -n "$1" ] && echo -n "$*" || echo "Failure"
    echo -en "\\033[0;39m"
}

upper() {
    echo "$*" | tr '[:lower:]' '[:upper:]'
}

echo "1. Installation ldap packages"
echo
echo

yum install openldap-servers openldap-clients

rm -rf /etc/openldap/slapd.d/
if [ ! -d /etc/openldap/slapd.d -a ! -f /etc/openldap/slapd.conf ]; then
    # convert from old style config slapd.conf
    mkdir -p /etc/openldap/slapd.d/
    slaptest -f /usr/share/openldap-servers/slapd.conf.obsolete -F /etc/openldap/slapd.d &>/dev/null
    chown -R ldap:ldap /etc/openldap/slapd.d
    chmod -R 000 /etc/openldap/slapd.d
    chmod -R u+rwX /etc/openldap/slapd.d
    rm -f /var/lib/ldap/*

    cp -af /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
    chown ldap.ldap /var/lib/ldap/DB_CONFIG
fi
echo
echo


echo
echo "2. syslog configuration"

FNAME="/etc/rsyslog.d/slapd.conf"
printf "   . %-60s ... " "${FNAME}"

mkdir -p /var/log/slapd >& /dev/null
chown -R ldap:ldap /var/log/slapd >& /dev/null

if [ ! -f "${FNAME}" ]; then
    cat > ${FNAME} <<EOF
local4.* /var/log/slapd/slapd.log
EOF
    if [ $? -eq 0 ]; then
        success "Done"
        echo
        /sbin/service rsyslog restart | sed 's/^/     /g'
    fi
else
    success "Pass"
    echo
fi

FNAME="/etc/logrotate.d/openldap"
printf "   . %-60s ... " "${FNAME}"

if [ ! -f "${FNAME}" ]; then
    cat > ${FNAME} <<EOF
/var/log/slapd/*.log {
    copytruncate
    rotate 4
    monthly
    notifempty
    missingok
    compress
    create 0644 ldap ldap
}
EOF
    [ $? -eq 0 ] && success "Done" || failure
else
    success "Pass"
fi
echo
echo


echo
echo "3. SLAPD configuration and start"

FNAME="/etc/sysconfig/ldap"
printf "   . %-60s ... " "${FNAME}"
perl -pi -e 's/#SLAPD_/SLAPD_/g' ${FNAME}

if [ $? -eq 0 ]; then
    success "Done"
    echo
    /sbin/service slapd restart | sed 's/^/     /g'
    [ -f /usr/bin/systemctl ] && \
        systemctl enable slapd || /sbin/chkconfig slapd on
else
    failure
    echo
fi
echo


echo "4. Order your BASE Information"

while [ -z "${opt}" ]
do
    echo -n "   Input your BASE DN : "
    read opt

    if [ -n "${opt}" ]; then
        echo "${opt}" | grep "dc=" >& /dev/null
        [ $? -ne 0 ] && opt=
    fi
done

BASEDN="${opt}"
BASEDN=$(echo ${BASEDN} | sed 's/ //g')
BASE=$(echo "$BASEDN" | perl -pe 's/,\s*dc=.*//g' | perl -pe 's/dc=/BASE=/g')
eval "${BASE}"
BASEUP="$(upper ${BASE})"

opt=""
while [ -z "${opt}" ]
do
    passwd_prompt "   Input your ldap administrator password   "
    opt=$prompt_value
    passwd_prompt "   Re-input your ldap administrator password"
    opt1=$prompt_value

    if [ -n "${opt}" -a -n "${opt1}" ]; then
        if [ "${opt}" != "${opt1}" ]; then
            echo "     .. passwords doesn't match!"
            opt=""
        fi
    else
        echo "     .. passwords doesn't match!"
        opt=""
    fi
done

ROOTPPW=${opt}
ROOTPW=$(slappasswd -s "${opt}")


echo
echo "   Reault:"
echo "           BASE           => ${BASE}"
echo "           BASE DN        => ${BASEDN}"
echo "           ADMIN Password => ${ROOTPPW}"
echo "           ADMIN Password => ${ROOTPW}"
echo



echo
echo "5. Settings BASE DN"

for file in /etc/openldap/slapd.d/cn\=config/olcDatabase*
do
    printf "   . %-60s ... " "${file}"
    perl -pi -e "s/dc=my-domain,dc=com/${BASEDN}/g" "$file"
    [ $? -eq 0 ] && success "Done" || failure
    echo
done

cat <<EOF | ldapmodify -Y EXTERNAL -H ldapi:/// >& /dev/null
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
echo


echo
echo "6. Settings Admin password"

cat <<EOF | ldapmodify -Y EXTERNAL -H ldapi:/// >& /dev/stdout | sed 's/^/   /g' | grep -v "^[ ]*$"
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: ${ROOTPW}

dn: olcDatabase={2}bdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: ${ROOTPW}
EOF

/sbin/service slapd restart | sed 's/^/     /g'
echo

echo
echo "7. Initial Directory Information Tree"

cat <<EOF | ldapadd -a -c -H ldapi:/// -D "cn=Manager,${BASEDN}" -w ${ROOTPPW} | sed 's/^/   /g' | grep -v "^[ ]*$"
dn: ${BASEDN}
dc: ${BASE}
o: ${BASEUP} LDAP
objectclass: dcObject
objectclass: organization
objectclass: top

dn: ou=Admin,${BASEDN}
ou: Users
objectclass: organizationalUnit

dn: ou=Users,${BASEDN}
ou: Users
objectclass: organizationalUnit

dn: ou=Groups,${BASEDN}
ou: Groups
objectclass: organizationalUnit
EOF

[ $? -eq 0 ] && success "Done" || failure
echo
echo



echo
echo "8. Create default groups"

cat <<EOF | ldapadd -a -c -H ldapi:/// -D "cn=Manager,${BASEDN}" -w ${ROOTPPW} | sed 's/^/   /g' | grep -v "^[ ]*$"
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

[ $? -eq 0 ] && success "Done" || failure
echo
echo

echo
echo "9. LDAP default Settings"

cat <<EOF | ldapmodify -Y EXTERNAL -H ldapi:/// | sed 's/^/   /g' | grep -v "^[ ]*$"
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

[ $? -eq 0 ] && success "Done" || failure
echo
echo

exit 0
[root@an3 ~]$
```

