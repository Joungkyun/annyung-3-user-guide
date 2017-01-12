# Slave Server 설정

>***목차***
>1. Slave daemon 준비
>2. Master data 준비
>3. Data restore
>


<br><br>

이 문서에서는 Muli-master-replication을 구축할 것이기 때문에, slave라 함은 나중에 셋팅이 되는 서버를 칭하는 용어가 되겠습니다. 그러니 굳이 Master/Slave에 연연해 하지 마시기 바랍니다. 그냥 여기서 말하는 Slave 서버 구축이라 함은, 복제 서버 구축 정도로 이해를 하는 것이 맞습니다.

##1. Slave daemon 준비

***Slave*** 의 설정은 ***Master***의 설정과 거의 동일합니다.

일단, [***Master*** Server 설정](chapter2-3-auth-intergrate-openldap-1.md)문서를 참고 하여 동일하게 설정을 합니다. (SSL 설정 까지 하였다면, SSL 설정 까지 동일하게 하십시오.)

설정 시의 <u>***Base DN***은 동일</u>해야 하며, LDAP manager의 암호는 관리 상의 문제이므로 동일하게 할지, 다르게 할지는 알아서 결정 하십시오.


##2. Master data 준비

***Master*** 서버에서 복제할 data를 ***dump*** 받습니다. 이 명령은 나중에 data backup용으로 사용을 할 수 있습니다.

```bash
[root@an3 ~]$ /usr/sbin/slapcat -l master-backup.ldif
5709f0f7 The first database does not allow slapcat; using the first available one (2)
# id=00000001
# id=00000002
# id=00000003
# id=00000004
# id=00000005
# id=00000006
# id=00000007
# id=00000008
# id=00000009
# id=0000000a
# id=0000000b
[root@an3 ~]$
```

이 데이터를 ***slave*** 서버의 ***/etc/openldap***으로 전송을 합니다. 여기서는 임의로 ***slave*** 서버를 ***an3-s*** 라 칭하겠습니다.

```bash
[root@an3 ~]$ rsync -av master-backup.ldif an3-s:/etc/openldap/
AnNyung release 3 (Labas)
Login an3-s.oops.org on an 15:22 on Sunday, 10 April 2016

Warning!! Authorized users only.
All activity may be monitored and reported

sending incremental file list
ldap.diff

sent 5510 bytes  received 31 bytes  3694.00 bytes/sec
total size is 5433  speedup is 0.98
[root@an3 ~]$
```

## 3. Data restore

***Master*** server에서 전체 DB 구조를 모두 backup을 받아왔기 때문에, 현재 ***Slave***에 있는 DB는 필요가 없습니다.

다음의 과정을 통하여 ***Master***의 DB를 ***Slave***에 restore 시키도록 합니다.

```bash
[root@an3-s ~]$ # 먼저 slapd를 종료 시킵니다.
[root@an3-s ~]$ service slapd stop
[root@an3-s ~]$ # Master의 data로 복구할 것이기 때문에 기존의 data를 삭제 합니다.
[root@an3-s ~]$ rm -rf /var/lib/ldap/*
[root@an3-s ~]$ # 기본 DB scheme를 복사 합니다.
[root@an3-s ~]$ cp -af /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@an3-s ~]$ # Master에서 backup한 data를 restore 합니다.
[root@an3-s ~]$ /usr/sbin/slapadd -l /etc/openldap/master-backup.ldif
5709f9e3 The first database does not allow slapadd; using the first available one (2)
_#################### 100.00% eta   none elapsed            none fast!
Closing DB...
[root@an3-s ~]$ # 생성된 data의 소유권을 변경하고, slapd daemon을 재시작 합니다.
[root@an3-s ~]$ chown ldap:ldap /var/lib/ldap/*
[root@an3-s ~]$ service slapd restart
slapd 를 정지 중:                                          [실패]
slapd (을)를 시작 중:                                      [  OK  ]
[root@an3-s ~]$
```

데이터를 restore를 했다면, 확인을 해 봅니다. 여기서는 restore된 ssoadmin 권한으로 Group OU를 탐색하도록 합니다. 만약, ***Master***에서 ssoadmin의 암호를 설정하지 않은 상태라면, ssoadmin 대신 LDAP 관리자 권한 (cn=manager,dc=oops.org,dc=org)으로 확인을 하십시오.

```bash
[root@an3 ~]$ # cn=manager,dc=oops,dc=org 권한으로 ou=Group,dc=oops,dc=org 의 entry 탐색
[root@an3 ~]$ ldapsearch -x -D "uid=ssoadmin,ou=admin,dc=oops,dc=org" -W -b "ou=Group,dc=oops,dc=org"
Enter LDAP Password: # ssoadmin 암호 입력
# extended LDIF
#
# LDAPv3
# base <ou=Group,dc=oops,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Group, oops.org
dn: ou=Group,dc=oops,dc=org
ou: Group
objectClass: organizationalUnit

# ldapusers, Group, oops.org
dn: cn=ldapusers,ou=Group,dc=oops,dc=org
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

data 복구가 완료가 되었으면, [Replaction 설정](chapter2-3-auth-intergrate-openldap-3.md) 문서를 참고하여 replication 설정을 하도록 합니다.