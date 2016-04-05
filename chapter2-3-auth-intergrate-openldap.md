# Openldap 인증 통합

현재 작성 중입니다. ___\*\\^^/\*___

openldap을 이용한 인증 통합은 openldap을 multi-master replaction 으로 구성을 하는 방법으로 기술을 합니다.

##1. Master server 설정

##1.1 package 설치

```bash
[root@an3 ~]$ yum install openldap-servers openldap-clients genpasswd
```

##1.2 