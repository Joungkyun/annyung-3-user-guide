# Sudo LDAP 연동


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
Defaults    env_keep =  "COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS"
Defaults    env_keep += "MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
Defaults    env_keep += "LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES"
Defaults    env_keep += "LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE"
Defaults    env_keep += "LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"

Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin

root    ALL=(ALL)   ALL
%wheel  ALL=(ALL)   ALL
EOF
[root@ldap1 ~]$
```

기본으로, ***root*** account와 ***%wheel*** group은 root권한으로 sudo를 실행할 수 있도록 한 설정 입니다. 이 설정을 ***ldap*** database에 추가 합니다.

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





```bash
[root@ldap1 ~]$ ldapadd -Y EXTERNAL -H ldapi:/// < sudo-ldap.txt
```