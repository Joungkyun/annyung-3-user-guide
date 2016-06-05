# LDAP data 관리

>***목차:***
>1. GUI Tool
2. Command line tool
  1. 계정 추가 및 삭제
  2. 계정 확인 및 속성(attribute) 변경
  3. 그룹 추가 및 삭제
  4. login host 제한 설정


##1. GUI Tool
console에서의 LDAP data관리는 너무나도 불편합니다. 특히나 설정 하나 변경을 하려면 ldif 파일을 만들어서 ldapadd 또는 ldapmodify 명령을 실행을 해야 한다는 것은 정말 짜증 나는 일입니다.

그렇기 때문에, Gui Tool들을 많이 사용하게 됩니다.

보통 openldap 관련 문서들을 보면 대부분 LDAP gui tool로서 Web base의 [phpldapadmin](http://phpldapadmin.sourceforge.net/wiki/index.php/Main_Page)을 소개하고 있으며, 그 외에서 디자인적으로도 화려한 java 기반의 gui tool들이 많이 있습니다.

하지만, 필자가 인증 통합을 목적으로 ldap을 구축하였을 때, 가장 편하게 사용을 할 수 있었던 tool은 [LdapAdmin](http://www.ldapadmin.org/)이라는 Gui tool을 권장 합니다. 다만 windows용 밖에 지원을 하지 않기 때문에 다른 OS에서는 아직 제가 테스트를 해 본 적이 없어 검색을 해 보셔서 마땅한 tool을 선택 하시기 바랍니다.

##2. Command line tool

앞에서 소개를 했듯이, ***ldap-auth-utils*** 패키지는 CLI(command line interface)에서 ldap을 관리하기 쉽게 하기 위하여 제공이 됩니다.

앞에서 소개를 했던 명령들 외에, ***ldap-auth-utils***는 다음의 명령들을 제공 합니다.

* ***ldap_auth*** : user 또는 group 속성 관리
* ***ldap_auth_init*** : LDAP database 인증 통합 초기화
* ***ldap_grpadd*** : LDAP group account 추가
* ***ldap_grpdel*** : LDAP group account 삭제
* ***ldap_host_manage*** : LDAP account host restrict 설정
* ***ldap_passwd.in*** : LDAP account passwd 변경
* ***ldap_replica*** : LDAP database replication 설정
* ***ldap_ssl*** : LDAP SSL 연결 설정
* ***ldap_useradd*** : LDAP user account 추가
* ***ldap_userdel*** : LDAP user account 삭제

### 1. 계정 추가 및 삭제

계정 추가/삭제는 ***ldap_useradd***와 ***ldap_userdel*** 명령을 이용합니다.

기본적인 사용법은 "***ldap_useradd [USER_ACCOUNT]***" 와 같이 합니다. ***USER_ACCOUNT***는 앞에서 사용했던 ***USER@BASEDN***의 형식을 사용하며, BASEDN은 domain 처럼 표현 합니다. 즉, ***DC=oops,DC=org***는 ***oops.org***와 같이 표현 합니다.

```shell
[root@ldap1 ~]$ ldap_useradd gildong.hong@oops.org
이름          : 길동 [성을 제외한 이름 입력]
성            : 홍
암호 입력                                : ***********
암호 재입력                              : ***********

Your informations:

    ID           : gildong.hong
    UID          : 10000
    GID          : 10000
    HOME         : /home/ldapusers/gildong.hong
    SHELL        : /bin/bash
    Expire Date  : 2243-10-19 00:00:00 (99999)
    SURNAME      : 홍
    NAME         : 길동
    GECOS        : LDAP Users
    GROUP Lists  :
    ACCEPT HOSTS :
    Last Changes : 2016-06-01 18:13:29 (16953)
    Passwd HASH  : {CRYPT}$1$Xwh8Ajhd$ogFBVAvOCY03qwFn.rKvh1

Is right your informations? [Y/N] : y
Regist account gildong.hong                ... OK
[root@ldap1 ~]$
```

기본적으로 아무런 옵션을 주지 않으면, 계정 이름과 암호 외의 정보는 ***/etc/openldap/ldap-auth-utils.conf***에 있는 값을 사용합니다. 다른 값을 변경 하고 싶으면 ***ldap_adduser -h*** 명령으로 옵션을 확인 하십시오.

```shell
[root@ldap1 ~]$ ladp_useradd -h
ldap_useradd: LDAP 데이터베이스에 user 추가
사용법: ldap_useradd [OPTIONS] USERNAME
옵션:
    -d HOME_DIR      홈 디렉토리 [기본값: /home/ldapusers/USERNAME]
    -e EXPIRE_DATE   계정 만료일 [기본값: 제한없음(0)]
                     Fromat is "YYYY-MM-DD HH:mm:SS" or unix timestamp
    -g GID           ID of the primary group [Default: ldapusers(10000)]
    -G GROUPS        list of supplementary groups
    -h               도움말 출력
    -H FQDN          host access privileges
    -i               interactive mode. ignore other options
    -n NAME          Real Name
    -l LAST NAME     Last Name
    -p PASSWORD      password. plain string or hashed string(with {CRYPT})
    -s SHELL         login shell [Default: /bin/bash]
    -u UID           user ID [Default: MAXUID + 1]
    -y               None intercative mode
    --gecos          Set GECOS field of passwd entry

USERNAME 형식
    형식 : ACCOUNT@DOMAIN_NAME

    if base dn of LDAP is "dc=DOMAIN,dc=COM", domain name is "DOMAIN.COM".

예제:
    # add LDAP_USER with BASE DN 'dc=DOMAIN,dc=COM'
    ldap_useradd LDAP_USER@DOMAIN.COM

    # 옵션으로 LDAP_USER를 등록
    ldap_useradd -n "Michael" -l "Jackson" LDAP_USER@DOMAIN.COM
    # add LDAP_USER with interactive mode
    ldap_useradd -i LDAP_USER@DOMAIN.COM
[root@ldap1 ~]$
```

계정 삭제는 간단하게 ***ldap_userdel*** 명령에 삭제할 계정만 지정을 하면 됩니다.

```shell
[root@ldap1 ~]$ ldap_userdel gildong.hong@oops.org
  * 'gildong.hong@oops.org' 계정을 삭제 하겠습니까? [yes/no]  : yes
    * 계정 삭제 gildong.hong@oops.org               ... OK
[root@ldap1 ~]$
```

### 2. 계정 확인 및 속성(attribute) 변경

존재하는 계정의 관리는 ***ldap_auth*** 명령을 이용 합니다. 일단 계정의 정보를 확인하기 위해서는 다음과 같이 실행을 합니다.

```shell
[root@ldap1 ~]$ ldap_auth gildong.hong@kldp.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=People,dc=oops,dc=org> with scope subtree
    # filter: (uid=gildong.hong)
    # requesting: ALL
    #
    # gildong.hong, People, oops.org
    compatibility dn : gildong.hong@oops.org
    dn               : uid=gildong.hong,ou=People,dc=oops,dc=org
    objectClass      : top
    objectClass      : inetOrgPerson
    objectClass      : posixAccount
    objectClass      : shadowAccount
    objectClass      : hostObject
    uid              : gildong.hong
    cn               : gildong.hong
    gecos            : LDAP Users
    givenName        : 길동
    sn               : 홍
    uidNumber        : 10000
    gidNumber        : 10000
    loginShell       : /bin/bash
    homeDirectory    : /home/ldapusers/gildong.hong
    shadowMin        : 0
    shadowMax        : 90
    shadowWarning    : 7
    shadowLastChange : 16953
    userPassword     : {CRYPT}$1$tiAx5gCv$8uwiBHCc3v6oRuT93gC.1/
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1

[root@ldap1 ~]$
```

그룹 account는 ***-g*** 옵션을 이용하여 확인 합니다.

```shell
[root@ldap1 ~]$ ldap_auth -g ldapusers@kldp.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=Group,dc=kldp,dc=org> with scope subtree
    # filter: (cn=ldapusers)
    # requesting: ALL
    #
    # ldapusers, Group, kldp.org
    compatibility dn : ldapusers@kldp.org
    dn               : cn=ldapusers,ou=Group,dc=kldp,dc=org
    objectClass      : posixGroup
    objectClass      : top
    cn               : ldapusers
    description      : LDAP account groups
    gidNumber        : 10000
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1

[root@ldap1 ~]$
```

확인 외에 attribute 값을 변경하거나 삭제할 수 있습니다. 이에 대해서는 help message 또는 man page를 참고 하십시오. (예제가 있습니다.) 단, 모든 attribute를 변경할 수 있지는 않습니다.

###3. 그룹 추가 및 삭제

그룹 추가/삭제는 ***ldap_grpadd***와 ***ldap_grpdel*** 명령을 사용합니다.

####1. 그룹 추가

DC=oops,DC=org에 ldapgrptest group 추가

```bash
[root@ldap1 ~]$ ldap_grpadd ldapgrptest@oops.org

Your informations:

    GROUP ID     : ldapgrptest
    GROUP DESC   : LDAP GROUP
    GID          : 10002
    MEMBERS      :

 * 'ldapgrptest'에 대한 그룹 엔트리 확인 .. 등록 진행 중
 ldapgrptest 그룹 엔트리 생성 .. OK
[root@ldap1 ~]$
```

####2. 그룹 멤버 추가

ldapgrptest group member로 gildong.hong account 추가

```bash
[root@ladp1 ~]$ ldap_grpadd -m gildong.hong ldapgrptest@oops.org

Your informations:

    GROUP ID     : ldapgrptest
    GROUP DESC   : LDAP GROUP
    GID          : 10002
    MEMBERS      : gildong.hong

 * 'ldapgrptest'에 대한 그룹 엔트리 확인 .. exsit
 * 추가 멤버 'gildong.hong' 확인 .. OK
 ldapgrptest에 추가 멤버 추가 .. OK
[root@ldap1 ~]$
```

####3. 그룹 멤버 삭제

ldapgrptest group에서 gildong.hong account 제거

```bash
[root@ladp1 ~]$ ldap_grpdel -m gildong.hong ldapgrptest@oops.org

Your informations:

    GROUP ID     : ldapgrptest
    MEMBERS      : gildong.hong

 * 'ldapgrptest'의 'gildong.hong' 삭제 .. OK
[root@ldap1 ~]$
```

####4. 그룹 삭제

cn=ldapgrptest,ou=Group,dc=oops,dc=org 그룹 삭제

```bash
[root@ladp1 ~]$ ldap_grpdel ldapgrptest@oops.org

Your informations:

    GROUP ID     : ldapgrptest
    MEMBERS      :

 * 'ldapgrptest' 그룹 삭제 .. OK
[root@ldap1 ~]$
```


###4. login host 제한 설정

####1. host entry 추가

***gildong.hong@oops.org*** account로 host1.oops.org와 host2.oops.org에 로그인 할 수 있도록 설정

```bash
[root@ldap1 ~]$ ldap_host_manager gildong.hong host1.oops.org
[root@ldap1 ~]$ ldap_host_manager gildong.hong host2.oops.org
[root@ldap1 ~]$ ldap_auth gildong.hong@kldp.org

    # extended LDIF
    #
    # LDAPv3
    # base <ou=People,dc=oops,dc=org> with scope subtree
    # filter: (uid=gildong.hong)
    # requesting: ALL
    #
    # gildong.hong, People, oops.org
    compatibility dn : gildong.hong@oops.org
    dn               : uid=gildong.hong,ou=People,dc=oops,dc=org
    objectClass      : top
    objectClass      : inetOrgPerson
    objectClass      : posixAccount
    objectClass      : shadowAccount
    objectClass      : hostObject
    uid              : gildong.hong
    cn               : gildong.hong
    gecos            : LDAP Users
    givenName        : 길동
    sn               : 홍
    uidNumber        : 10000
    gidNumber        : 10000
    loginShell       : /bin/bash
    homeDirectory    : /home/ldapusers/gildong.hong
    shadowMin        : 0
    shadowMax        : 90
    shadowWarning    : 7
    shadowLastChange : 16953
    userPassword     : {CRYPT}$1$tiAx5gCv$8uwiBHCc3v6oRuT93gC.1/
    host             : host1.oops.org
    host             : host2.oops.org
    # search result
    search           : 3
    result           : 0 Success
    # numResponses: 2
    # numEntries: 1

[root@ldap1 ~]$
```

####2. host entry 제가
