# Replication 설정

> ***목차***
> 1. Relication user 설정
> 2. Replication 설정
> 3. replication 확인
> 4. Multi-Msater Replication 유의 사항

여기서의 replication 설정은 multi master 설정을 하게 됩니다. 즉, replication으로 구성된 모든 서버가 master이자 slave가 된다는 의미 입니다.

그러므로, 설정 자체는 동일하게 되면, target server만 달라지게 됩니다.

##1. Relication user 설정

"***2.3.1.1 Master Server 설정***" 문서에서 이미 repliaction을 위하여 ***replica(uid=replica,ou=Admin,dc=kldp,dc=org)***라는 account를 생성해 놓았습니다.

***replica*** account는 ***ldapROusers*** Group에 포함이 되어 있기 때문에, ldap 전체 database에 대한 읽기 권한을 가지게 됩니다.

이미 account는 준비가 되었으므로, 여기서는 ***replica*** account의 password만 설정을 하도록 합니다.

다만, replication 설정시의 account 암호는 평문(plain/text)로 설정을 하기 때문에, 될 수 있으면 난수와 비슷한 암호를 사용하는 것을 권장 합니다.

***master***에서 다음의 명령을 실행 합니다.

```bash
[root@an3 ~]$ export BASEDN="dc=oops,dc=org"
[root@an3 ~]$ ldappasswd -x -D "cn=manager,${BASEDN}"  "uid=replica,ou=admin,${BASEDN}" -W
Enter LDAP Password:  # master의 LDAP 관리자 암호 입력
New password: 6dGU6sD/
[root@an3 ~]$ 
```

위와 같이 ldappasswd 명령을 ***-S*** 옵션 없이 실행을 하면, ramdom password를 생성해서 등록해 줍니다. 저 암호를 잘 보관한 다음 ***slave***에서 동일하게 등록을 합니다.

***-s*** 옵션으로 암호를 명령행으로 전달을 할 수 있습니다.

```bash
[root@an3-s ~]$ export BASEDN="dc=oops,dc=org"
[root@an3-s ~]$ ldappasswd -x -D "cn=manager,${BASEDN}" -s "6dGU6sD/" "uid=replica,ou=admin,${BASEDN}" -W
Enter LDAP Password:  # slave의 LDAP 관리자 암호 입력
[root@an3-s ~]$ 
```

##2. Replication 설정

적당한 디렉토리에 replica-add.ldif와 replica-modify.ldif라는 2개의 ldif 파일을 아래와 생성 합니다. (변경이 필요한 부분에는 주석을 달아 놓았습니다. ldapadd와 ldapmodify로 등록을 할 때는 주석이 있으면 안되니, 제거하십시오.)

```bash
[root@an3 ~]$ cat replica-add.ldif
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la

dn: olcOverlay=syncprov,olcDatabase={2}bdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
[root@an3 ~]$
[root@an3 ~]$ cat replica-modify.ldif
dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 0

dn: olcDatabase={2}bdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001
  # 동기화 시킬 서버를 지정합니다.
  provider=ldaps://an3-s.oops.org/
  bindmethod=simple
  # dc=oops,dc=org 부분은 설정한 Base DN으로 변경
  binddn="uid=replica,ou=Admin,dc=oops,dc=org"
  # replica account의 암호를 평문으로 등록
  credentials=6dGU6sD/
  # dc=oops,dc=org 부분은 설정한 Base DN으로 변경
  searchbase="dc=oops,dc=org"
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry="30 5 300 3"
  interval=00:00:05:00
-
add: olcMirrorMode
olcMirrorMode: TRUE

dn: olcOverlay=syncprov,olcDatabase={2}bdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
[root@an3 ~]$
```

ldif 파일이 준비가 되었으며, master와 slave 모두 등록을 합니다. ***replica-add.ldif***는 ***ldapadd*** 명령으로 등록을 하고, ***replica-modify.ldif***는 ***ldapmodify*** 명령을 이용하여 등록 합니다.

```bash
[root@an3 ~]$ ldapadd -Y EXTERNAL -H ldapi:/// -f ./replica-add.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=module,cn=config"

adding new entry "olcOverlay=syncprov,olcDatabase={2}bdb,cn=config"

[root@an3 ~]$ ldapmodify -Y EXTERNAL -H ldapi:/// -f ./replica-modify.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"

modifying entry "olcDatabase={2}bdb,cn=config"

adding new entry "olcOverlay=syncprov,olcDatabase={2}bdb,cn=config"

[root@an3 ~]$
```

***slave*** 에서는 ***master***에서 사용한 ldif 파일 중, replica-modify.ldif에서 ***provider*** 설정만 ***master*** 서버로 등록을 해 주면 됩니다.

##3. replication 확인

***ssoadmin*** account의 암호를 변경해 보도록 합니다.

먼저 master와 slave의 ssoadmin account의 현재 userPassword object를 확인 합니다.

자신의 암호 외에 다른 account의 암호 변경은 LDAP 관리자와 ssoadmin에게만 있으며, 현재 ssoadmin의 암호가 없는 상태이기 때문에 LDAP 관리자의 권한으로 실행을 하도록 합니다.

```bash
[root@an3 ~]$ # master(an3) 계정 확인
[root@an3 ~]$ ldapsearch -H ldaps:/// -D "cn=manager,dc=oops,dc=org" -b "dc=oops,dc=org" "(uid=ssoadmin)" -W | grep userPassword
Enter LDAP Password:    # LDAP 관리자 암호 입력(cn=manager)
[root@an3 ~]$ # slave(an3-s) 계정 확인
[root@an3 ~]$ ldapsearch -H ldaps://an3-s.oops.org/ -D "cn=manager,dc=oops,dc=org" -b "dc=oops,dc=org" "(uid=ssoadmin)" -W | grep userPassword
Enter LDAP Password:    # LDAP 관리자 암호 입력(cn=manager)
[root@an3 ~]$
```

***Master(an3.oops.org)***와 ***Slave(an3-s.oops.org)*** 모두 현재 ***userPassword*** object가 존재하지 않는 상태 입니다.

먼저 ***Master***에서 ***ssoadmin*** account의 암호를 변경 합니다.

```bash
[root@an3 ~]$ ldappasswd -H ldaps:/// -x -D "cn=manager,dc=oops,dc=org" -S "uid=ssoadmin,ou=admin,dc=oops,dc=org" -W
New password:            # 변경할 암호 입력
Re-enter new password:   # 변경할 암호 재 입력
Enter LDAP Password:     # cn=manager(LDAP 관리자)의 암호 입력
[root@an3 ~]$
```

암호가 변경이 되었는지, 그리고 ***Slave***로 replication이 되었는지 ***Master***와 ***Slave***의 ssoadmin account entry를 확인해 봅니다.

```bash
[root@an3 ~]$ # master(an3) 계정 확인
[root@an3 ~]$ ldapsearch -H ldaps:/// -D "cn=manager,dc=oops,dc=org" -b "dc=oops,dc=org" "(uid=ssoadmin)" -W | grep userPassword
Enter LDAP Password:    # LDAP 관리자 암호 입력(cn=manager)
userPassword:: e1NTSEF9RW80NE4KZ2NLbXNyNDdmeFTNN3RvRVRmbUNrdlFFZTc=
[root@an3 ~]$ # slave(an3-s) 계정 확인
[root@an3 ~]$ ldapsearch -H ldaps://an3-s.oops.org/ -D "cn=manager,dc=oops,dc=org" -b "dc=oops,dc=org" "(uid=ssoadmin)" -W | grep userPassword
Enter LDAP Password:    # LDAP 관리자 암호 입력(cn=manager)
userPassword:: e1NTSEF9RW80NE4KZ2NLbXNyNDdmeFTNN3RvRVRmbUNrdlFFZTc=
[root@an3 ~]$
```

***Slave(an3-s.oops.org)***에도 ***userPassword*** object가 생성이 된 것이 확인이 됩니다. 그럼 반대로 ***Slave(an3-s.oops.org)***에서 변경을 했을 경우, ***master(an3.oops.org)***에서도 변경이 되는 지 확인 해 봅니다.

```bash
[root@an3 ~]$ ldappasswd -H ldaps://an3-s.oops.org/ -x -D "cn=manager,dc=oops,dc=org" -S "uid=ssoadmin,ou=admin,dc=oops,dc=org" -W
New password:            # 변경할 암호 입력
Re-enter new password:   # 변경할 암호 재 입력
Enter LDAP Password:     # cn=manager(LDAP 관리자)의 암호 입력
[root@an3 ~]$
```

암호가 변경이 되었는지, 그리고 ***Master***로 replication이 되었는지 ***Master***와 ***Slave***의 ssoadmin account entry를 확인해 봅니다.

```bash
[root@an3 ~]$ # master(an3) 계정 확인
[root@an3 ~]$ ldapsearch -H ldaps:/// -D "cn=manager,dc=oops,dc=org" -b "dc=oops,dc=org" "(uid=ssoadmin)" -W | grep userPassword
Enter LDAP Password:    # LDAP 관리자 암호 입력(cn=manager)
userPassword:: e1NTSEF9SE16ZlQ5c0RFMEh4NGZadnRKbTNtYjBEWktTM1VTaww=
[root@an3 ~]$ # slave(an3-s) 계정 확인
[root@an3 ~]$ ldapsearch -H ldaps://an3-s.oops.org/ -D "cn=manager,dc=oops,dc=org" -b "dc=oops,dc=org" "(uid=ssoadmin)" -W | grep userPassword
Enter LDAP Password:    # LDAP 관리자 암호 입력(cn=manager)
userPassword:: e1NTSEF9SE16ZlQ5c0RFMEh4NGZadnRKbTNtYjBEWktTM1VTaww=
[root@an3 ~]$
```

역시 변경된 것을 확인할 수 있습니다.

##4. Multi-Msater Replication 유의 사항

openldap의 replication에서 주의할 점은, network 단절이나 server down이 발생할 경우, 이 동안 업데이트 된 것에 대한 양측 데이터의 정합성을 보장하지 못합니다.

즉, network 단절이나 server down이 발생할 경우, 수동으로 data를 맞추어 주든지 또는 한쪽을 기준으로 slave 설정을 다시 해야 한다는 의미 입니다.