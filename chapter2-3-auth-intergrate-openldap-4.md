# Replication 설정

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

