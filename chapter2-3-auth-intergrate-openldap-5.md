# LDAP 인증 연동 설정

>***목차***
>1. 연동에 필요한 인증 설정
>2. 필요 package 설치
>3. 인증 연동 설정
>4. 연동 확인
>

<br><br>

설명을 하기 전에 다음의 상황으로 간주를 하오니, 예제의 shell prompt를 잘 살피시기 바랍니다.

1. Master server ***ldap1.oops.org***
2. Slave server ***ldap2.oops.org***
3. Client ***an3.oops.org***

앞 장에서 설명을 했듯이, ***Multi-Master Replication***으로 LDAP 서버를 구성했기 때문에, 명칭 상 Master와 Slave를 구분한 것이지, 기능적으로는 둘 다 Master 입니다.


## 1. 연동에 필요한 인증 설정

LDAP 인증을 시작하기 전에, LDAP server에서 연동에 사용할 account 설정을 합니다.

LDAP 설정 초기화 시에, 연동에 필요한 ***ssomanager***라는 account를 미리 생성해 놓았으며, ***ssomanager*** account는 ***ldapROusers*** 그룹의 member로 생성이 되었습니다.

또한, ***ldapROusers*** 그룹은 LDAP 전체 database에 대해서 read only 권한을 가지고 있습니다.

앞에서 암호를 미리 지정해 놓았습니다만 혹시 잊어 버렸거나 또는 설정을 하지 않았다면 ***master(ldap1.oops.org)*** 또는 ***slave(ldap2.oops.org)***에서 다음 명령을 이용하여 다시 설정을 합니다. (replication 설정이 되어 있으므로 어디서 해도 상관이 없습니다.)

```bash
[root@ldap1 ~]$ ldap_passwd -u Admin ssomanager@oops.org
New password     : ***********
Re-New password  : ***********

Your Informations:

    * Account: ssoadmin@oops.org
    * RDN : uid=ssomanager,ou=Admin,dc=oops,dc=org
    * Host: ldapi:///
    * Privilieges: -Y EXTERNAL
    * Commnad: /usr/bin/ldapmodify -H "ldapi:///" -Y EXTERNAL
    * Hash: {CRYPT}$6$htk01t9cUA5aFM/a$9H4.kig058cRESS6MGdjn8armHHP6IAQO9Qykr6iroW9laqugz.bIOPNzBUgk8N4H01QkeklEwQg05FBzSrfz/

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "uid=ssoadmin,ou=Admin,dc=oops,dc=org"

0
Done
[root@an3 ~]$
```


## 2. 필요 패키지 설치

LDAP 연동을 할 서버(LDAP client server, 여기서는 ***an3*** host입니다.)에서 ***nss-pam-ldapd*** package와 ***nss-pam_ldap*** package를 설치합니다.

```bash
[root@an3 ~]$ yum install openldap-clients nss-pam-ldapd
```

***RHEL/CentOS 6***과 ***안녕 2***에서는 ***pam_ldap*** package가 의존성 설정에 의해 같이 설치가 됩니다.

## 3. 인증 연동 설정

먼저, LDAP 서버 구성 시에 SSL을 가능하도록 하였다면, 인증서의 CA 인증서를 클라이언트에 복사 합니다. self sign 인증서를 사용한다면, CA 인증서는 system에 있는 ***/etc/pki/tls/certs/ca-bundle.crt***를 이용하시면 되고, 발급 기관에서 받았을 경우에는 CA에서 제공하는 CA 인증서를 이용 합니다.

다음은 ***startssl***의 CA 인증서를 예로 듭니다.

```bash
[root@an3 ~]$ rsync -av \
     ldap1:/etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem \
     /etc/openldap/certs/
[root@an3 ~]$ chmod 644 /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
[root@an3 ~]$
```

다음의 명령으로 시스템을 LDAP에 연동 합니다.

```bash
[root@an3 ~]$ authconfig --enableldap \
                       --enableldapauth \
                       --ldapserver="ldaps://ldap1.oops.org ldaps://ldap2.oops.org" \
                       --ldapbasedn="dc=oops,dc=org" \
                       --enablemkhomedir \
                       --update
[root@an3 ~]$
```

다음 ***/etc/openldap/ldap.conf***와 ***/etc/nslcd.conf*** 에서 다음의 값을들 확인 합니다. 다르면 수정을 하고, 설정이 안되어 있으면 추가해 주도록 합니다.

```bash
[root@an3 ~]$ # 먼저 /etc/openldap/ldap.conf 를 먼저 설정 합니다.
[root@an3 ~]$ cat /etc/openldap/ldap.conf
TLS_CACERTDIR /etc/openldap/cacerts
TLS_CACERT /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
TLS_REQCERT allow

URI ldaps://ldap1.oops.org/ ldaps://ldap2.oops.org
BASE dc=oops,dc=org
[root@an3 ~]$
[root@an3 ~]$ # 다음은, /etc/nslcd.conf 를 설정 합니다.
[root@an3 ~]$
[root@an3 ~]$ cat /etc/nslcd.conf
uid nslcd
gid ldap

# SSL 설정
ssl no
tls_cacertdir /etc/openldap/cacerts
tls_cacertfile /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem

# LDAP servers
uri ldaps://ldap1.oops.org/ ldaps://ldap2.oops.org/

# 인증 정보
binddn uid=ssomanager,ou=admin,dc=oops,dc=org
# LDAP 서버에서 설정한 ssomanager의 암호를 평문으로 작성
bindpw 평문암호

# database biding
base dc=oops,dc=org
# Admin database를 출력하지 않도록 제한을 함
base   group  ou=Group,dc=oops,dc=org
base   passwd ou=People,dc=oops,dc=org
base   shadow ou=People,dc=oops,dc=org
[root@an3 ~]$ chmod 600 /etc/nslcd.conf
[root@an3 ~]$
[root@an3 ~]$ # nslcd 재시작
[root@an3 ~]$ service nslcd restart
nslcd 를 정지 중:                                          [실패]
nslcd (을)를 시작 중:                                      [  OK  ]
[root@an3 ~]$ systemctl enable nslcd
[root@an3 ~]$
```

RHEL 6/CentOS 6/안녕 2에서는 ***/etc/pam_ldap.conf***의 다음 설정 값들을 수정/추가 하십시오.

```bash
[root@an2 ~]$ cat /etc/pam_ldap.conf
# The distinguished name of the search base.
base dc=oops,dc=org

# Another way to specify your LDAP server is to provide an
# uri with the server name. This allows to use
# Unix Domain Sockets to connect to a local LDAP Server.
uri ldaps://ldap1.oops.org/ ldaps://ldap2.oops.org/

# OpenLDAP SSL mechanism
# start_tls mechanism uses the normal LDAP port, LDAPS typically 636
ssl no
tls_cacertdir /etc/openldap/cacerts
tls_cacertfile /etc/pki/startssl/startssl-sub.class2.server.ca.sha2.pem

# RFC2307bis naming contexts
# Syntax:
# nss_base_XXX      base?scope?filter
# where scope is {base,one,sub}
# and filter is a filter to be &'d with the
# default filter.
# You can omit the suffix eg:
# nss_base_passwd   ou=People,
# to append the default base DN but this
# may incur a small performance impact.
nss_base_passwd    ou=People,dc=example,dc=com?one
nss_base_shadow    ou=People,dc=example,dc=com?one
nss_base_group     ou=Group,dc=example,dc=com?one

# password hashing
# md5로 지정을 하면 md5 crypt, sha512 crypt를 이용하여 로그인이 가능 합니다.
# {SSHA}는 로그인이 안됩니다.
pam_password md5
[root@an2 ~]
```


여기까지 하면, 기본적으로 연동이 완료 되었습니다. 

## 4. 연동 확인

먼저, ***pwck***와 ***grpck*** 명령으로 /etc/passwd 와 /etc/group, /etc/shadow를 정리 합니다. (uid, gid 순서로 sort를 합니다.)

```
[root@an3 ~]$ pwck -s && grpck -s
[root@an3 ~]$
```

다음 시스템 계정 정보를 확인 합니다.

```bash
[root@an3 ~]$ getent passwd
root:x:0:0:root:/root:/bin/bash
   ... 중략 ...
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
saslauth:x:499:76:"Saslauthd user":/var/empty/saslauth:/sbin/nologin
[root@an3 ~]$ getent group
root:x:0:
  ... 중략 ...
nobody:x:99:
users:x:100:
ldapusers:*:10000:
[root@an3 ~]$
```

***OU=People***에 추가된 account가 없기 때문에 passwd list에서는 ldap account가 보이지 않지만, group list에서는 ***OU=Group***에 생성한 ***ldapusers*** group을 확인할 수 있습니다.

그럼, master server에서 ***ldapuser1*** 이라는 account를 추가해 보도록 하겠습니다.

```bash
[root@ladp1 ~]$ cat > ldapuser1.ldif <<EOF
dn: uid=ldapuser1,ou=People,dc=oops,dc=org
objectClass: posixAccount
objectClass: top
objectClass: inetOrgPerson
objectClass: shadowAccount
uidNumber: 10002
gidNumber: 10000
givenName: 이름
sn: 성
uid: ldapuser1
homeDirectory: /home/ldapuser/ldapuser1
gecos: LDAP user 1
loginShell: /bin/bash
shadowFlag: 0
shadowMin: 0
shadowMax: 99999
shadowWarning: 0
shadowInactive: 99999
shadowExpire: 99999
cn: ldapuser1
userPassword: {CRYPT}$1$ORDoH6WC$b5T.3AUpf1eICJVTRIPzO1
shadowLastChange: 16903
EOF
[root@ldap1 ~]$ ldapadd -x -D cn=manager,dc=oops,dc=org -W -f ldapuser1.ldif
Enter LDAP Password: # LDAP 관리자 암호 입력
adding new entry "uid=ldapuser1,ou=People,dc=oops,dc=org"
[root@ldap1 ~]$
```

일단, 위의 정보를 그대로 입력해서 테스트 유저를 만듭니다. 위의 예제의 암호는 "***asdf***"를 md5 crypt 한 것입니다. 유저 생성에 대해서는 뒤에서 따로 다룰 예정 입니다.

다음, 연동한 client에서 다시 확인을 해 봅니다.

```bash
[root@an3 ~]$ # password entry 를 확인 합니다.
[root@an3 ~]$ getent passwd | grep ldapuser1
ldapuser1:x:10001:10000:"LDAP user 1":/home/ldapuser/ldapuser1:/bin/bash
[root@an3 ~]$ # shadow entry 를 확인 합니다.
[root@an3 ~]$ getent shadow | grep ldapuser1
ldapuser1:$1$ORDoH6WC$b5T.3AUpf1eICJVTRIPzO1:16903:0:99999:0:99999:99999:0
[root@an3 ~]$
```

마지막으로 login 을 테스트 해 봅니다.

```bash
[root@an3 ~]$ ssh ldapuser1@localhost
AnNyung LInux 3 (Labas)
Login an3.oops.org on an 01:32 on Wednesday, 13 April 2016

Warning!! Authorized users only.
All activity may be monitored and reported

ldapuser1@localhost''s password:
Creating directory '/home/ldapuser/ldapuser1'.
[ldapuser1@an3 ~]$
```