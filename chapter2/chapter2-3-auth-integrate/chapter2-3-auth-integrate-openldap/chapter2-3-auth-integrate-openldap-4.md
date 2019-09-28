# Replication 설정

> _**목차**_ 1. Replication 설정 2. replication 확인 3. Multi-Msater Replication 유의 사항

여기서의 replication 설정은 multi master 설정을 하게 됩니다. 즉, replication으로 구성된 모든 서버가 master이자 slave가 된다는 의미 입니다.

그러므로, 설정 자체는 동일하게 되며, target server만 달라지게 됩니다.

## 1. Relication 설정

"_**2.3.1.1 Master Server 설정**_" 문서에서 _**ldap\_auth\_init**_를 이용하여 초기화 했을 때, 이미 repliaction을 위하여 _**replica\(uid=replica,ou=Admin,dc=oops,dc=org\)**_라는 account를 생성해 놓았며, 암호 설정을 해 놓았습니다.

_**replica**_ account는 _**ldapROusers**_ Group에 포함이 되어 있기 때문에, ldap 전체 database에 대한 읽기 권한을 가지게 됩니다.

이미 account는 준비가 되었고, replcation 설정을 위하여 _**ldap\_replica**_라는 명령을 이용합니다.

```bash
[root@an3 ~]$ ldap_replica
이 서버는 replication 설정이 되어 있지 않습니다!
[root@an3 ~]$
```

아무런 옵션 없이 _**ldap\_replica**_를 실행하면 replication 설정을 보여주게 됩니다. 현재는 설정이 안되어 있기 때문에 위 처럼 메시지가 나와야 합니다.

replication 설정은 _**-a**_ 옵션을 이용하며, 제거는 _**-r**_ 옵션을 이용합니다.

위에서 설명을 했듯이 _**master**_, _**slave**_ 구분이 의미가 없지만, 여기서는 서버의 구분을 위하여 _**master**_를 _**ldap1.oops.org**_, _**slave**_를 _**ldap2.oops.org**_라고 호칭하면서 설명을 진행하도록 합니다.

_**master\(ldap1.oops.org\)**_에서 _**slave\(ldap2.oops.org\)**_의 변경 사항을 반영하도록 설정 합니다.

```bash
[root@ldap1 ~]$ ldap_replica -a -i 0 -u uid=replica,ou=Admin,dc=oops,dc=org ldap2.oops.org
Input replica password : ********* [replica account 계정 암호 입력]
설정 정보:

    Replica Account         : uid=replica,ou=admin,dc=oops,dc=org
    Replica BIND DN         : dc=oops,dc=org
    Replica Pre Test        : OK
    Replica Server ID       : 0
    Replica Provider        : ldap2.oops.org

이 정보가 맞습니까? [Y/N] : y

 * 1. sync 모듈 등록 ...  OK
 * 2. Set Server ID ... OK
 * 3. 리플리케이션 서버 설정 ... OK
 * 4. Mirror 모드 설정 ... OK
 * 5. 리플리케이션 오버레이 설정 ... OK

설정 완료

[root@ldap1 ~]$
```

다음, _**slave\(ldap2.oops.org\)**_에서 _**master\(ldap1.oops.org\)**_의 변경 사항을 반영하도록 설정 합니다. 주의할 것은 _**master\(ldap1.oops.org\)**_와 _**Replica Server ID\(-i 옵션\)**_ 값을 다르게 설정 하십시오.

```bash
[root@ldap2 ~]$ ldap_replica -a -i 1 -u uid=replica,ou=Admin,dc=oops,dc=org ldap1.oops.org
Input replica password : ********* [replica account 계정 암호 입력]
설정 정보:

    Replica Account         : uid=replica,ou=admin,dc=oops,dc=org
    Replica BIND DN         : dc=oops,dc=org
    Replica Pre Test        : OK
    Replica Server ID       : 1
    Replica Provider        : ldap1.oops.org

이 정보가 맞습니까? [Y/N] : y

 * 1. sync 모듈 등록 ...  OK
 * 2. Set Server ID ... OK
 * 3. 리플리케이션 서버 설정 ... OK
 * 4. Mirror 모드 설정 ... OK
 * 5. 리플리케이션 오버레이 설정 ... OK

설정 완료

[root@ldap2 ~]$
```

## 2. replication 확인

_**ssoamanager**_ account의 암호를 변경해 보도록 합니다.

먼저 _**ldap\_auth**_ 명령을 이용하여 _**master**_에서 _**ssomanager**_의 정보를 확인 합니다.

```bash
[root@ldap1 ~]$ ldap_auth -o Admin ssomanager@oops.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=Admin,dc=oops,dc=org> with scope subtree
    # filter: (uid=ssomanager)
    # requesting: ALL
    #
    # ssomanager, Admin, oops.org
    compatibility dn : ssomanager@oops.org
    dn               : uid=ssomanager,ou=Admin,dc=oops,dc=org
    objectClass      : posixAccount
    objectClass      : top
    objectClass      : inetOrgPerson
    objectClass      : shadowAccount
    gidNumber        : 9997
    givenName        : SSO
    sn               : Manager
    displayName      : SSO Manager
    uid              : ssomanager
    homeDirectory    : /
    gecos            : SSO Manager
    loginShell       : /sbin/nologin
    shadowFlag       : 0
    shadowMin        : 0
    shadowMax        : 99999
    shadowWarning    : 0
    shadowInactive   : 99999
    shadowLastChange : 12011
    shadowExpire     : 99999
    cn               : SSO manager
    uidNumber        : 9998
    userPassword     : {CRYPT}$6$htk01t9cUA5aFM/a$9H4.kig058cRESS6MGdjn8armHHP6IAQO9Qykr6iroW9laqugz.bIOPNzBUgk8N4H01QkeklEwQg05FBzSrfz/
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1

[root@ldap1 ~]$
```

_**master\(ldap1.oops.org\)**_에서 _**ssomanager**_의 암호를 변경합니다.

```bash
[root@ldap1 ~]$ ldap_passwd -u admin ssomanager@oops.org
New password     : **************
Re-New password  : **************

Your Informations:

    * Account: ssomanager@oops.org
    * RDN : uid=ssomanager,ou=Admin,dc=oops,dc=org
    * Host: ldapi:///
    * Privilieges: -Y EXTERNAL
    * Commnad: /usr/bin/ldapmodify -H "ldapi:///" -Y EXTERNAL
    * Hash: {CRYPT}$6$qeB3N9K2FgIMds0m$tHCwJ3wS37zlDvgeza3T7ddtZN1Pc5s9qs0ROHsbPXE5MoFaQA3I8Uu.vEgcSjOSyd3/zqS.g6d/FHt2YEHf.0

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "uid=ssomanager,ou=Admin,dc=oops,dc=org"

0
Done
[root@ldap1 ~]$
```

_**slave\(ldap2.oops.org\)**_에서 _**ssomanager**_ account를 확인하여 _**userPassword**_ 값이 _**master\(ldap1.oops.org\)**_와 동일한지 확인 합니다.

```bash
[root@ldap2 ~]$ ldap_auth -o Admin ssomanager@oops.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=Admin,dc=oops,dc=org> with scope subtree
    # filter: (uid=ssomanager)
    # requesting: ALL
    #
    # ssomanager, Admin, oops.org
    compatibility dn : ssomanager@oops.org
    dn               : uid=ssomanager,ou=Admin,dc=oops,dc=org
    objectClass      : posixAccount
    objectClass      : top
    objectClass      : inetOrgPerson
    objectClass      : shadowAccount
    gidNumber        : 9997
    givenName        : SSO
    sn               : Manager
    displayName      : SSO Manager
    uid              : ssomanager
    homeDirectory    : /
    gecos            : SSO Manager
    loginShell       : /sbin/nologin
    shadowFlag       : 0
    shadowMin        : 0
    shadowMax        : 99999
    shadowWarning    : 0
    shadowInactive   : 99999
    shadowLastChange : 12011
    shadowExpire     : 99999
    cn               : SSO manager
    uidNumber        : 9998
    userPassword     : {CRYPT}$6$qeB3N9K2FgIMds0m$tHCwJ3wS37zlDvgeza3T7ddtZN1Pc5s9qs0ROHsbPXE5MoFaQA3I8Uu.vEgcSjOSyd3/zqS.g6d/FHt2YEHf.0
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1

[root@ldap2 ~]$
```

변경사항이 반영이 확인이 되었다면, 반대로 테스트를 해 봅니다.

## 3. Multi-Msater Replication 유의 사항

openldap의 replication에서 주의할 점은, network 단절이나 server down이 발생할 경우, 이 동안 업데이트 된 것에 대한 양측 데이터의 정합성을 보장하지 못합니다.

즉, network 단절이나 server down이 발생할 경우, 수동으로 data를 맞추어 주든지 또는 한쪽을 기준으로 slave 설정을 다시 해야 한다는 의미 입니다.

또한, 서버의 replication 설정은 slave 설정이므로, ldap1의 변경 사항을 sync 하지 않으려면 ldap2의 replication 설정을 제거해야 합니다.

