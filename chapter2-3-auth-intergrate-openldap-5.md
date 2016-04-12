# Client Ldap 연동 설정

>***목차***
>1. 연동에 필요한 인증 설정
>2. 필요 package 설치
>3. 인증 연동 설정
>3. nslcd 설정

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
[root@an3 ~]$ yum install openldap-clients nss-pam-ldapd pam_ldap
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
```


다음의 명령으로 시스템을 LDAP에 연동 합니다.

```bash
[root@an3 ~]$ authconfig --enableldap \
                       --enableldapauth \
                       --enablemkhomedir \
                       --update 
```