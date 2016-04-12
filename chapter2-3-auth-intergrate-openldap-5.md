# Client Ldap 인증 연동 설정

>***목차***
>1. 필요 package 설치
>2. authconfig를 이용한 인증 연동
>3. nslcd 설정

<br><br>

설명을 하기 전에 다음의 상황으로 간주를 하오니, 예제의 shell prompt를 잘 살피시기 바랍니다.

1. Master server ***ldap1***
2. Slave server ***ldap2***
3. Client ***an3***

앞 장에서 설명을 했듯이, ***Multi-Master Replication***으로 LDAP 서버를 구성했기 때문에, 명칭 상 Master와 Slave를 구분한 것이지, 기능적으로는 둘 다 Master 입니다.


## 1. 필요 패키지 설치

```bash
[root@an3 ~]$ yum install nss-pam-ldapd pam_ldap
```
