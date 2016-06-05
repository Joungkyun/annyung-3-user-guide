# Sudo LDAP 연동

>*** 목차***
>1. 기본 설정
>2. sudo entry 추가
>3. LDAP client 설정 및 확인

<br><br>

이번 장은 SUDO를 LDAP을 이용하여 연동하는 방법을 기술 합니다.

***ldap_auth_init***를 이용하여 LDAP을 초기화 하였다면, 이미 ***Openldap***에서 ***sudo***를 연동하기 위한 준비는 마친 상태 입니다. 혹시 이 과정에 관심이 있다면 ***ldap-auth-utils*** package의 ***ldap_auth_init*** script를 분석해 보든지 또는 http://www.potatogim.net/wiki/OpenLDAP/Sudo 문서를 참고 하십시오.

## 1. 기본 설정

***sudo***를 연동하기 위한 준비는 마친 상태라고 위에서 기술 하였지만, 현재 ***ldap*** database에는 ***sudo***에 대한 아무런 data가 존재하지 않는 상태 입니다. 그러므로 여기서는 기본 data를 입력하는 과정을 기술 합니다.

일단, ldap 서버에서 ***ldap***에 입력하기 위한 기본 파일을 작성 합니다.

```bash
[root@ ldap1 ~]$ cat <<<EOF > sudo-ldap.txt
Defaults    requiretty
Defaults    !visiblepw
Defaults    always_set_home

Defaults    env_reset
Defaults    env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS"
Defaults    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"

Defaults    secure_path=/sbin:/bin:/usr/sbin:/usr/bin

root    ALL=(ALL)   ALL
%wheel  ALL=(ALL)   ALL
EOF
[root@ldap1 ~]$
```

기본으로, ***root*** account와 ***%wheel*** group은 root 권한으로 sudo를 실행할 수 있도록 한 설정 입니다. 이 설정을 ***ldap*** database에 추가 합니다.

먼저 sudo-ladp.txt 파일을 ***LDIF***로 변환 합니다.

```bash
[root@an3 ~]$ cat <<EOF
dn: ou=sudo,dc=oops,dc=org
objectClass: top
objectClass: organizationalUnit
description: sudo
ou: sudo

EOF
[root@an3 ~]$ sudo SUDOERS_BASE="ou=sudo,dc=oops,dc=org" perl $(rpm -ql sudo | grep sudoers2ldif) ./sudo-ldap.txt >> sudo-ldap.ldif
[root@an3 ~]$ 
```

위의 예제를 참고하면 짐작할 수 있듯이, /etc/sudoers 파일을 직접 ***sudoers2ldif*** 필터로 LDIF로 변환할 수도 있습니다. 여기서 주의해야 하는 것이, ***Defaults***의 값에서 ***env_keep***이나 ***secure_path***의 값 처럼 "=" 또는 "+=" 앞뒤로 공백이 있을 경우 "***sudo: unknown defaults entry 'env_keep '***"와 같은 에러가 발생하니 주의해야 합니다.

변환한 ***LDIF*** 파일을 이용하여 ***ldap*** database에 입력을 합니다.

```bash
[root@ldap1 ~]$ ldapadd -Y EXTERNAL -H ldapi:/// < sudo-ldap.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "ou=sudo,dc=oops,dc=org"

adding new entry "cn=defaults,ou=sudo,dc=oops,dc=org"

adding new entry "cn=root,ou=sudo,dc=oops,dc=org"

adding new entry "cn=%wheel,ou=sudo,dc=oops,dc=org"

[root@ldap1 ~]$
```

잘 입력이 되었는지 확인을 해 봅니다.

```bash
[root@ldap1 ~]$ ldapsearch -Y EXTERNAL -H ldapi:/// -b ou=sudo,dc=oops,dc=org
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
# extended LDIF
#
# LDAPv3
# base <ou=sudo,dc=oops,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# sudo, oops.org
dn: ou=sudo,dc=oops,dc=org
objectClass: top
objectClass: organizationalUnit
description: sudo
ou: sudo

# defaults, sudo, oops.org
dn: cn=defaults,ou=sudo,dc=oops,dc=org
objectClass: top
objectClass: sudoRole
cn: defaults
description: Default sudoOption''s go here
sudoOption: requiretty
sudoOption: !visiblepw
sudoOption: always_set_home
sudoOption: env_reset
sudoOption: env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS"
sudoOption: env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
sudoOption: env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
sudoOption: env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
sudoOption: env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
sudoOption: secure_path=/sbin:/bin:/usr/sbin:/usr/bin
sudoOrder: 1

# root, sudo, oops.org
dn: cn=root,ou=sudo,dc=oops,dc=org
objectClass: top
objectClass: sudoRole
cn: root
sudoUser: root
sudoHost: ALL
sudoRunAsUser: ALL
sudoCommand: ALL
sudoOrder: 2

# %wheel, sudo, oops.org
dn: cn=%wheel,ou=sudo,dc=oops,dc=org
objectClass: top
objectClass: sudoRole
cn: %wheel
sudoUser: %wheel
sudoHost: ALL
sudoRunAsUser: ALL
sudoCommand: ALL
sudoOption: !authenticate
sudoOrder: 3

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 4
[root@ldap1 ~]$
```

## 2. sudo entry 추가

sudo entry에서 사용하는 attribute는 다음과 같습니다.

```ruby
sudoUser: root
sudoHost: ALL
sudoRunAsUser: ALL
sudoCommand: ALL
sudoOption: authenticate
sudoOrder: 3
```

여기서는 sudotest 라는 account가 an3.oops.org 에서 sudo를 이용하여 an3.oops.org에서 인증없이 ls 명령을 실행하는 권한을 예제로 합니다.

먼저, ***ldap_useradd*** 명령을 이용하여, sudotest ldap account를 생성합니다.

일단 먼저 sudoOrder의 최대값을 체크 합니다.

```bash
[root@ldap1 ~]$ ldapsearch -Y EXTERNAL -H ldapi:/// -b ou=sudo,dc=oops,dc=org "(sudoOrder=*)" sudoOrder 2> /dev/null | awk -F ": " '/^sudoOrder:/ {print $2}' | sort -r | head -n 1
3
[root@ldap1 ~]$
```

다음, 아래와 같이 ldif 파일을 생성 한 후, ***ldap*** database에 추가 합니다. sudoOrder 값은 위에서 구한 "최대값 + 1" 로 지정을 합니다.

```bash
[root@ldap1 ~]$ cat <<EOF > add-sudotest.txt
dn: cn=sudotest,ou=sudo,dc=oops,dc=org
objectClass: top
objectClass: sudoRole
cn: sudotest
sudoUser: sudotest
sudoHost: an3.oops.org
sudoRunAsUser: ALL
sudoCommand: /bin/ls
sudoOption: !authenticate
sudoOrder: 4
EOF
[root@ldap1 ~]$ ldapadd -Y EXTERNAL -H ldapi:/// < add-sudotest.txt
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=sudotest,ou=sudo,dc=oops,dc=org"

[root@ldap1 ~]$
```

클라이언트에서의 동작 확인은 [2.3.1.6 LDAP 클라이언트 인증 연동](chapter2-3-auth-intergrate-openldap-6.md) 문서에서 다루도록 하겠습니다.

## 3. LDAP client 설정 및 확인

다음 설정은 ldap client인 ***an3*** host에서 합니다.

###1. sudo-ldap.conf 설정

***/etc/sudo-ldap.conf***에 LDAP 연동 설정을 해 줍니다. ***/etc/nslcd.conf***의 설정을 참조하면 됩니다.

```bash
[root@an3 ~]$ cat <<EOF >> /etc/sudo-ldap.conf
base           dc=oops,dc=org
uri            ldaps://ldap1.oops.org ldaps://ldap2.oops.org
binddn         uid=ssomanager,ou=admin,dc=oops,dc=org
bindpw         ssomanager__평문암호
TLS_CACERTFILE /etc/openldap/certs/pki/1_root_bundle.crt
tls_checkpeer  no
timelimit      15
sudoers_base   ou=sudo,dc=oops,dc=org
[root@an3 ~]$
```

###2. nsswitch.conf 설정

***/etc/nsswitch.conf*** 에 ***sudoers*** 설정을 추가 합니다.

```bash
[root@an3 ~]$ echo "sudoers:    files ldap" >> /etc/nsswitch.conf
[root@an3 ~]$
```

nscd databse를 갱신해 주도록 합니다.

```bash
[root@an3 ~]$ service nscd force-reload
[root@an3 ~]$
```

###3. ldap sudo 연동 테스트

위에서 추가한 sudotest account로 테스트를 진행해 봅니다.

```bash
[root@an3 ~]$ su - sudotest
'/home/ldapusers/sudotest' 디렉토리 생성 중.
[sudotest@an3 ~]$ sudo ls /root
anaconda-ks.cfg  ks-post.log
[sudotest@an3 ~]$
```