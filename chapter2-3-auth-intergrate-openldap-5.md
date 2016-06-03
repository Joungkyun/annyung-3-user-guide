# LDAP 인증 연동

>***목차***
>1. 연동에 필요한 인증 설정
>2. 필요 package 설치
>3. 인증 연동 설정
>4. 연동 확인
>5. host attribute를 이용한 로그인 제한
>6. sudo 권한 설정 (Not yet)
>

<br><br>

설명을 하기 전에 다음의 상황으로 간주를 하오니, 예제의 shell prompt를 잘 살피시기 바랍니다.

1. Master server ***ldap1.oops.org***
2. Slave server ***ldap2.oops.org***
3. Client
 * RHEL 7 계열: ***an3.oops.org***
 * RHEL 6 계열: ***an2.oops.org***

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

LDAP 연동을 할 서버(LDAP client server, 여기서는 ***an3*** host입니다.)에서 ***nss-pam-ldapd*** package와 ***ldap-auth-utils-passwd*** package를 설치합니다.

```bash
[root@an3 ~]$ yum install openldap-clients nss-pam-ldapd ldap-auth-utils-passwd
```

***RHEL 6***계열에서는 ***pam_ldap*** package가 의존성 설정에 의해 같이 설치가 됩니다.

***openldap-clients***, ***nss-pam-ldapd***는 LDAP과 연동을 하는데 사용이 되어지며, ***ldap-auth-utils-passwd*** 패키지는 system에서 LDAP database의 암호를 변경하기 위하여 사용을 합니다. 물론 ***openldap-clients***의 ***ldappasswd***를 이용해도 상관 없지만, ldif 통신을 해야 하는 귀찮음으로 사용하기 쉬운 ***ldap_passwd*** 명령을 제공 합니다.

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

다음 ***/etc/openldap/ldap.conf***와 ***/etc/nslcd.conf*** 에서 다음의 값들을 확인 합니다. 다르면 수정을 하고, 설정이 안되어 있으면 추가해 주도록 합니다.

먼저 인증 정보가 들어가기 때문에 파일의 권한 설정을 해 주도록 합니다.

```bash
[root@an3 ~]$ chmod 600 /etc/nslcd.conf
```

***RHEL 6*** 계열은 /etc/pam_ladp.conf 도 수정을 해 줍니다.

```bash
[root@an2 ~]$ chmod 600 /etc/nslcd.conf /etc/pam_ldap.conf
```

다음, ***/etc/openldap/ldap.conf***와 ***/etc/nslcd.conf***의 설정을 확인 합니다.

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
[root@an3 ~]$
[root@an3 ~]$ # nslcd 재시작
[root@an3 ~]$ service nslcd restart
nslcd 를 정지 중:                                          [실패]
nslcd (을)를 시작 중:                                      [  OK  ]
[root@an3 ~]$
```

Booting 시에 nslcd가 동작하도록 설정 합니다.

***RHEL 7*** 계열에서는

```bash
[root@an3 ~]$ systemctl enable nslcd
```

***RHEL 6*** 계열에서는 다음과 같이 합니다.
```bash
[root@an3 ~]$ chkconfig nslcd on
```

***RHEL/CentOS 6***, ***AnNyung 2***에서는 ***/etc/pam_ldap.conf***도 설정을 해 줘야 합니다. ***/etc/pam_ldap.conf***의 다음 설정 값들도 수정/추가 하십시오.

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
nss_base_passwd    ou=People,dc=oops,dc=org?one
nss_base_shadow    ou=People,dc=oops,dc=org?one
nss_base_group     ou=Group,dc=oops,dc=org?one

# password hashing
# md5로 지정을 하면 md5 crypt, sha512 crypt를 이용하여 로그인이 가능 합니다.
# {SSHA}는 로그인이 안됩니다.
pam_password md5

# BIND DN
# 인증 정보
binddn uid=ssomanager,ou=admin,dc=oops,dc=org
# LDAP 서버에서 설정한 ssomanager의 암호를 평문으로 작성
bindpw 평문암호
[root@an2 ~]$
```

다음, nscd를 구동하고 있는 시스템이라면 <u>nscd database를 갱신</u>해 줘야 합니다. (안그러면 30분 정도 기다려야 할 수도 있습니다.)

```bash
[root@an3 ~]$ service nscd force-reload
```

여기까지 하면, 기본적으로 연동이 완료 되었습니다. 

## 4. 연동 확인

먼저, ***pwck***와 ***grpck*** 명령으로 /etc/passwd 와 /etc/group, /etc/shadow를 정리 합니다. (uid, gid 순서로 sort를 합니다.)

```
[root@an3 ~]$ pwck -s && grpck -s
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
[root@ldap1 ~]$ ldap_useradd ldapuser1@oops.org
ldap_useradd  ldapuser1@kldp.org
이름          : 이름
성            : 성
암호 입력                                : ***********
암호 재입력                              : ***********

Your informations:

    ID           : ldapuser1
    UID          : 10002
    GID          : 10000
    HOME         : /home/ldapusers/ldapuser1
    SHELL        : /bin/bash
    Expire Date  : 2243-10-19 00:00:00 (99999)
    SURNAME      : 성
    NAME         : 이름
    GECOS        : LDAP Users
    GROUP Lists  :
    ACCEPT HOSTS :
    Last Changes : 2016-06-03 20:00:02 (16955)
    Passwd HASH  : {CRYPT}$6$xTSXCyCXbBF12xwo$er4hbzAjFJKTueScqg1UT.msN1w0TY4EauaBtoBJ4Eeb2Oy/ZDNbf8O7hlkffroLPMY4c1njTDKncH/pxU5ob/

Is right your informations? [Y/N] : y
Regist account ldapuser1                   ... OK
[root@ldap1 ~]$ ldap_auth ldapuser1@oops.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=People,dc=oops,dc=org> with scope subtree
    # filter: (uid=ldapuser1)
    # requesting: ALL
    #
    # ldapuser1, People, kldp.org
    compatibility dn : ldapuser1@kldp.org
    dn               : uid=ldapuser1,ou=People,dc=kldp,dc=org
    objectClass      : top
    objectClass      : inetOrgPerson
    objectClass      : posixAccount
    objectClass      : shadowAccount
    objectClass      : hostObject
    uid              : ldapuser1
    cn               : ldapuser1
    gecos            : LDAP Users
    givenName        : 엘댑
    displayName      : 김 엘댑
    sn               : 김
    uidNumber        : 10002
    gidNumber        : 10000
    loginShell       : /bin/bash
    homeDirectory    : /home/ldapusers/ldapuser1
    shadowMin        : 0
    shadowMax        : 90
    shadowWarning    : 7
    shadowLastChange : 16955
    userPassword     : {CRYPT}$6$xTSXCyCXbBF12xwo$er4hbzAjFJKTueScqg1UT.msN1w0TY4EauaBtoBJ4Eeb2Oy/ZDNbf8O7hlkffroLPMY4c1njTDKncH/pxU5ob/
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1

[root@an3 ~]$
```

일단, 위와 같이 테스트 유저를 만듭니다. 다음, 연동한 client에서 다시 확인을 해 봅니다.

```bash
[root@an3 ~]$ # password entry 를 확인 합니다.
[root@an3 ~]$ getent passwd | grep ldapuser1
ldapuser1:x:10002:10000:"LDAP user":/home/ldapusers/ldapuser1:/bin/bash
[root@an3 ~]$ # shadow entry 를 확인 합니다.
[root@an3 ~]$ getent shadow | grep ldapuser1
ldapuser1:*:16955:0:90:7:::0
[root@an3 ~]$
```

참고로, ***getent shadow***에서 암호 필드는 ***RHEL 7*** 호환 계열은 위와 같이 ___*___로 masking이 되어서 나오고, ***RHEL 6*** 호환 이하로는 hashed string으로 출력이 됩니다.

마지막으로 login 테스트를 해 봅니다.

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

##5. host attribute를 이용한 로그인 제한

***ldap-auth-utils***를 이용하여 LDAP 인증 연동을 구성할 경우, 기본으로 ldap entry의 host attribute를 이용하여 로그인을 제한 할 수 있도록 구성되어 있습니다.

일단, 이 기능을 사용하기 위해서는 ldap client에서 다음 설정을 해야 합니다.

***RHEL 7*** 호환 계열:
```bash
[root@an3 ~]$ echo 'pam_authz_search (&(objectClass=posixAccount)(uid=$username)(|(host=$hostname)(host=$fqdn)(host=\\*)))' >> /etc/nslcd.conf
[root@an3 ~]$ service nslcd restart
```

***RHEL 6*** 호환 계열:
```bash
[root@an3 ~]$ echo "pam_check_host_attr yes" >> /etc/pam_ldap.conf
[root@an3 ~]$ service nslcd restart
```

설정을 마쳤으면 다신 ***ldapuser1*** account로 로그인을 해 봅니다.

```bash
[root@an3 ~]$ ssh ldapuser1@localhost
AnNyung LInux 3 (Labas)
Login an3.oops.org on an 21:18 on Friday, 03 June 2016

Warning!! Authorized users only.
All activity may be monitored and reported

ldapuser1@localhost's password:
LDAP authorisation check failed
Connection closed by 127.0.0.1
[root@an3 ~]$
```

아까는 로그인이 되었지만, host 설정 이후, ldapuser1 entry에 host attribute가 없기 때문에 로그인이 되지 않습니다. ***secure*** log는 다음과 같이 남습니다.

```bash
[root@an3 ~]$ tail /var/log/secure
Jun  3 20:41:50 open sshd[9570]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=127.0.0.1  user=ldapuser1
Jun  3 20:41:50 open sshd[9570]: pam_ldap(sshd:account): LDAP authorisation check failed; user=ldapuser1; err=Success
Jun  3 20:41:50 open sshd[9570]: Failed password for ldapuser1 from 127.0.0.1 port 33566 ssh2
Jun  3 20:41:50 open sshd[9570]: fatal: Access denied for user ldapuser1 by PAM account configuration [preauth]
Jun  3 21:18:50 open sshd[11938]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=127.0.0.1  user=ldapuser1
Jun  3 21:18:50 open sshd[11938]: pam_ldap(sshd:account): LDAP authorisation check failed; user=ldapuser1; err=Success
Jun  3 21:18:50 open sshd[11938]: Failed password for ldapuser1 from 127.0.0.1 port 33619 ssh2
Jun  3 21:18:50 open sshd[11938]: fatal: Access denied for user ldapuser1 by PAM account configuration [preauth]
[root@an3 ~]$
```

그럼, login이 가능하도록 ldapuser1 entry에 host attribute 설정을 해 보도고 합니다.

***master(ldap1.oops.org)***나 ***slave(ldap2.oops.org)***에서 다음의 명령을 실행 합니다.

```bash
[root@ldap1 ~]$ ldap_host_manage ldapuser1@oops.org an3.oops.org
[root@ldap1 ~]$ ldap_auth ldapuser1@oops.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=People,dc=oops,dc=org> with scope subtree
    # filter: (uid=ldapuser1)
    # requesting: ALL
    #
    # ldapuser1, People, kldp.org
    compatibility dn : ldapuser1@kldp.org
    dn               : uid=ldapuser1,ou=People,dc=kldp,dc=org
    objectClass      : top
    objectClass      : inetOrgPerson
    objectClass      : posixAccount
    objectClass      : shadowAccount
    objectClass      : hostObject
    uid              : ldapuser1
    cn               : ldapuser1
    gecos            : LDAP Users
    givenName        : 엘댑
    displayName      : 김 엘댑
    sn               : 김
    uidNumber        : 10002
    gidNumber        : 10000
    loginShell       : /bin/bash
    homeDirectory    : /home/ldapusers/ldapuser1
    shadowMin        : 0
    shadowMax        : 90
    shadowWarning    : 7
    shadowLastChange : 16955
    userPassword     : {CRYPT}$6$xTSXCyCXbBF12xwo$er4hbzAjFJKTueScqg1UT.msN1w0TY4EauaBtoBJ4Eeb2Oy/ZDNbf8O7hlkffroLPMY4c1njTDKncH/pxU5ob/
    host             : an3.oops.org
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1
[root@ldap1 ~]$
```

***ldap_auth*** 결과를 보면, password hash 문자열 아래에 ***host*** attribute가 설정이 되어 있음을 확인할 수 있습니다. ***ldap_host_manage*** 명령을 이용하여 추가 또는 삭제를 할 수 있습니다.

주의할 것은, host 이름은 해당 서버의 ***FQDN***으로 설정을 해 줘야 합니다.

이제 다시, 로그인 테스트를 해 보도록 합니다.

```bash
[root@an3 ~]$ ssh ldapuser1@localhost
AnNyung LInux 3 (Labas)
Login an3.oops.org on an 21:32 on Friday, 03 June 2016

Warning!! Authorized users only.
All activity may be monitored and reported

ldapuser1@localhost''s password:
Last failed login: Fri Jun  3 21:18:50 KST 2016 from 127.0.0.1 on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Fri Jun  3 20:41:29 2016 from 127.0.0.1
[ldayuser1@an3 ~]$
```

이제 로그인이 잘 되는 것을 확인할 수 있습니다.

## 6. sudo 권한 설정