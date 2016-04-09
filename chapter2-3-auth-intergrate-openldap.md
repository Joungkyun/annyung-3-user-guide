# Openldap 인증 통합

현재 작성 중입니다. ___\*\\^^/\*___

openldap을 이용한 인증 통합은 openldap을 multi-master replaction 으로 구성을 하는 방법으로 기술을 합니다.

##1. Master server 설정

##1.1 package 설치

```bash
[root@an3 ~]$ yum install openldap-servers openldap-clients genpasswd
```

##1.2 Openldap 초기화

***openldap***은 ***splapd*** daemon을 이용하여 구동이 됩니다. 또한, 2.4.23 버전 부터는 ***slapd.conf*** 대신에 ***OLC(OnLineConfiguration, cn=config 구조)***로 변경이 되었습니다.

물론 기존의 ***slapd.conf***를 migration 해 주는 방법을 제공하고 있기는 하나, 여기서는 그냥 ***OLC***를 이용하는 방법으로 설명을 합니다. ***AnNyung 2*** 에서도 동일하게 ***OLC*** 방식으로 설정을 해야 합니다. (RHEL 6 부터 ***OLC***방식으로 변경이 되었습니다.)


###1.2.1 OLC 초기화

일단, 처음 ***openldap-servers*** package를 설치 하신 경우 또는, openldap 설정을 한번도 안한 상태라면, 이 단계는 뛰어 넘어도 상관이 없습니다.

이 단계는, 기존의 openldap 설정을 모두 날려 버리고, 설치 시의 상태로 복원 시키는 것이니, 이 작업을 할지에 대해서는 신중하게 판단을 하십시오.

```bash
[root@an3 ~]$ # slapd.conf와 slapd.d 디렉토리를 제거 합니다.
[root@an3 ~]$ rm -f /etc/openldap/slapd.conf
[root@an3 ~]$ rm -rf /etc/openldap/slapd.d
[root@an3 ~]$ # slapd.d 를 생성하고 초기화 합니다.
[root@an3 ~]$ mkdir -p /etc/openldap/slapd.d
[root@an3 ~]$ slaptest -f /usr/share/openldap-servers/slapd.conf.obsolete \
                     -F /etc/openldap/slapd.d
[root@an3 ~]$ # slapd.d 의 디렉토리 파일의 권한에서 group/extra 권한을 모두 제거 합니다.
[root@an3 ~]$ chown -R 000 /etc/openldap/slapd.d
[root@an3 ~]$ chmod -R u+rwX /etc/openldap/slapd.d
[root@an3 ~]$ #  기존의 openldap data를 모두 초기화 시킵니다.
[root@an3 ~]$ rm -rf /var/lib/ldap/*
[root@an3 ~]$ cp -af /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@an3 ~]$ chown ldap.ldap /var/lib/ldap/DB_CONFIG
```