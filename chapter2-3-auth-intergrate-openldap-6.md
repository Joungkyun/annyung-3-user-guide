# LDAP data 관리

>***목차:***
>1. GUI Tool
2. Command line tool
  1. 계정 추가 및 삭제


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
[root@an3 ~]$ ldap_useradd gildong.hong@oops.org
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
[root@an3 ~]$
```

기본적으로 아무런 옵션을 주지 않으면, 계정 이름과 암호 외의 정보는 ***/etc/openldap/ldap-auth-utils.conf***에 있는 값을 사용합니다. 다른 값을 변경 하고 싶으면 ***ldap_adduser -h*** 명령으로 옵션을 확인 하십시오.

```shell
[root@an3 ~]$ ladp_useradd -h
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
[root@an3 ~]$
```

계정 삭제는 간단하게 ***ldap_userdel*** 명령에 삭제할 계정만 지정을 하면 됩니다.

```shell
[root@an3 ~]$ ldap_userdel gildong.hong@kldp.org
  * 'gildong.hong@kldp.org' 계정을 삭제 하겠습니까? [yes/no]  : yes
    * 계정 삭제 gildong.hong@kldp.org               ... OK
[root@an3 ~]$
```

