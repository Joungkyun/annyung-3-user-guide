# Replication 설정

> ***목차***
> 1. Replication 설정
> 2. replication 확인
> 3. Multi-Msater Replication 유의 사항

<br><br>

여기서의 replication 설정은 multi master 설정을 하게 됩니다. 즉, replication으로 구성된 모든 서버가 master이자 slave가 된다는 의미 입니다.

그러므로, 설정 자체는 동일하게 되며, target server만 달라지게 됩니다.

##1. Relication 설정

"***2.3.1.1 Master Server 설정***" 문서에서 ***ldap_auth_init***를 이용하여 초기화 했을 때, 이미 repliaction을 위하여 ***replica(uid=replica,ou=Admin,dc=oops,dc=org)***라는 account를 생성해 놓았며, 암호 설정을 해 놓았습니다.

***replica*** account는 ***ldapROusers*** Group에 포함이 되어 있기 때문에, ldap 전체 database에 대한 읽기 권한을 가지게 됩니다.

이미 account는 준비가 되었고, replcation 설정을 위하여 ***ldap_replica***라는 명령을 이용합니다.

```shell
[root@an3 ~]$ ldap_replica
이 서버는 replication 설정이 되어 있지 않습니다!
[root@an3 ~]$
```

아무런 옵션 없이 ***ldap_replica***를 실행하면 replication 설정을 보여주게 됩니다. 현재는 설정이 안되어 있기 때문에 위 처럼 메시지가 나와야 합니다.

replication 설정은 ***-a*** 옵션을 이용하며, 제거는 ***-r*** 옵션을 이용합니다.

위에서 설명을 했듯이 ***master***, ***slave*** 구분이 의미가 없지만, 여기서는 서버의 구분을 위하여 ***master***를 ***ldap1.oops.org***, ***slave***를 ***ldap2.oops.org***라고 호칭하면서 설명을 진행하도록 합니다.

***master(ldap1.oops.org)***에서 ***slave(ldap2.oops.org)***의 변경 사항을 반영하도록 설정 합니다.

```shell
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

다음, ***slave(ldap2.oops.org)***에서 ***master(ldap1.oops.org)***의 변경 사항을 반영하도록 설정 합니다. 주의할 것은 ***master(ldap1.oops.org)***와 ***Replica Server ID(-i 옵션)*** 값을 다르게 설정 하십시오.

```shell
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


##2. replication 확인

***ssoamanager*** account의 암호를 변경해 보도록 합니다.

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