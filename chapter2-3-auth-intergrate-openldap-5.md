# Client Ldap 연동 설정

>***목차***
>1. 연동에 필요한 인증 설정
>2. 필요 package 설치
>3. 인증 연동 설정
>4. 연동 확인
>

<br><br>

설명을 하기 전에 다음의 상황으로 간주를 하오니, 예제의 shell prompt를 잘 살피시기 바랍니다.

1. Master server ***ldap1***
2. Slave server ***ldap2***
3. Client ***an3***

앞 장에서 설명을 했듯이, ***Multi-Master Replication***으로 LDAP 서버를 구성했기 때문에, 명칭 상 Master와 Slave를 구분한 것이지, 기능적으로는 둘 다 Master 입니다.


## 1. 연동에 필요한 인증 설정

LDAP 인증을 시작하기 전에, LDAP server에서 연동에 사용할 account 설정을 합니다.

LDAP 서버 구축 시에, 연동에 사용을 하기 위하여 ***ssomanager***라는 account를 미리 생성해 놓았드벼, ***ssomanager*** 는 ***ldapROusers*** 그룹의 member로 생성이 되었습니다.

또한, ***ldapROusers*** 그룹은 LDAP 전체 database에 대해서 read only 권한을 가지고 있습니다.

이, ***ssomanager*** account의 암호를 설정 하도록 합니다.

Master 서버 (ldap1)에서 LDAP 관리자의 권한(***cn=manager,dc=oops,dc=org***)으로 설정을 합니다. Slave(ldap2)에서 해도 무방합니다. (어차피 Multi matster 구성이기 때문입니다.)

```bash
[root@ldap1 ~]$ export BASEDN="dc=oops,dc=org"
[root@ldap1 ~]$ ldappasswd -x -D "cn=manager,${BASEDN}"  "uid=ssomanager,ou=admin,${BASEDN}" -W -S
New password:            # 변경할 암호 입력
Re-enter new password:   # 변경할 암호 재 입력
Enter LDAP Password:     # cn=manager(LDAP 관리자)의 암호 입력
[root@an3 ~]$
```

## 2. 필요 패키지 설치

LDAP 연동을 할 서버(LDAP client server, 여기서는 ***an3*** host입니다.)에서 ***nss-pam-ldapd*** package와 ***pam_ldap*** package를 설치합니다.

```bash
[root@an3 ~]$ yum install openldap-clients nss-pam-ldapd
```

## 3. 인증 연동 설정

먼저, LDAP 서버 구성시에 SSL을 가능하도록 하였다면, 인증서의 CA 인증서를 클라이언트에 복사 합니다. CA 인증서는 앞에서 예로 든 ***startssl***의 공인 인증서를 예로 듭니다. self sign을 하셨거나 다른 공인 인증서를 사용하신다면, 사용하는 인증서의 CA 인증서를 복사 하시면 됩니다.

```bash
[root@an3 ~]$ rsync -av \
     ldap1:/etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem \
     /etc/openldap/certs/
[root@an3 ~]$ chmod 644 /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
[root@an3 ~]$
```

다음 ***/etc/openldap/ldap.conf***와 ***/etc/nslcd.conf*** 에 LDAP 기본 설정을 하도록 합니다.

```bash
[root@an3 ~]$ # 먼저 /etc/openldap/ldap.conf 를 먼저 설정 합니다.
[root@an3 ~]$ cat >> /etc/openldap/ldap.conf <<EOF
TLS_CACERT /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
URI ldaps://ldap1.oops.org/
URI ldaps://ldap2.oops.org/
BASE dc=oops,dc=org
EOF
[root@an3 ~]$
[root@an3 ~]$ # 다음은, /etc/nslcd.conf 를 설정 합니다.
[root@an3 ~]$
[root@ane ~]$ # 기본 설정을 제거 합니다.
[root@an3 ~]$ perl -pi -e 's/^(uid|gid|uri|base)[\s]+/#$1 /g' /etc/nslcd.conf
[root@an3 ~]$ # 필요한 설정을 추가 합니다.
[root@an3 ~]$ cat >> /etc/nslcd.conf <<EOF
# SSL 설정
ssl no
tls_cacertdir /etc/openldap/cacerts
tls_cacertfile /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem

# LDAP servers
uri ldaps://ldap1.oops.org/
uri ldaps://ldap2.oops.org/

# 인증 정보
binddn uid=ssomanager,ou=admin,dc=oops,dc=org
# LDAP 서버에서 설정한 ssomanager의 암호를 평문으로 작성
bindpw 평문암호

# database biding
base dc=oops,dc=org
# Admin database를 출력하지 않도록 제한을 함
base   group  ou=Groups,dc=oops,dc=org
base   passwd ou=Users,dc=oops,dc=org
base   shadow ou=Users,dc=oops,dc=org
EOF
[root@an3 ~]$ chmod 600 /etc/nslcd.conf
[root@an3 ~]$
[root@an3 ~]$ # nslcd 재시작
[root@an3 ~]$ service nslcd restart
nslcd 를 정지 중:                                          [실패]
nslcd (을)를 시작 중:                                      [  OK  ]
[root@an3 ~]$ systemctl enable nslcd
[root@an3 ~]$
```

다음의 명령으로 시스템을 LDAP에 연동 합니다.

```bash
[root@an3 ~]$ authconfig --enableldap \
                       --enableldapauth \
                       --enablemkhomedir \
                       --update
[root@an3 ~]$
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

***OU=Users***에 추가된 account가 없기 때문에 passwd list에서는 ldap account가 보이지 않지만, group list에서는 ***OU=Groups***에 생성한 ***ldapusers*** group을 확인할 수 있습니다.

그럼, master server에서 ***ldapuser1*** 이라는 account를 추가해 보도록 하겠습니다.

```bash
[root@ladp1 ~]$ cat > ldapuser1.ldif <<EOF
dn: uid=ldapuser1,ou=Users,dc=oops,dc=org
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
adding new entry "uid=ldapuser1,ou=Users,dc=oops,dc=org"
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