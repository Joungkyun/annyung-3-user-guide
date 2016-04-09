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
EOF  # 여기까지 실행명령

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
modifying entry "olcDatabase={2}bdb,cn=config"
[root@an3 ~]$
```


