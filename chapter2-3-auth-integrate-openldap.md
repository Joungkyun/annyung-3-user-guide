# Openldap 인증 통합

OpenLDAP을 이용한 통합 인증을 위하여 안녕 리눅스는 ***ldap-auth-utils***라는 패키지를 지원합니다.

보통 ldap을 효율적으로 관리를 하기에는 ***openldap***의 command line 도구들이 ldap과의 binary 통신을 위하여 ldif format을 사용하기 때문에 상당히 불편하기 때문에 보통은 GUI tool을 사용합니다.

하지만, GUI tool들도 특성이 너무 많고, LDAP의 사용 범위 역시 인증 통합에 국한이 된 것이 아니기 때문에 GUI tool
도 선택을 하기가 쉽지 않습니다.

그래서 안녕 리눅스는, command line에서 ldap 인증을 관리하기 위한 ***ldap-auth-utils*** frontend package를 지원 하며, 이를 이용하여 integartion을 기술 합니다.

openldap을 이용한 인증 통합은 openldap을 multi-master replaction 으로 구성을 하는 방법으로 기술을 합니다.

이 문서는 RHEL >= 6, CentOS >= 6, AnNyung >=2 에 적용이 가능 합니다.

1. [Master Server 설정](chapter2-3-auth-integrate-openldap-1.md)
2. [SSL 설정](chapter2-3-auth-integrate-openldap-2.md)
3. [Slave Server 설정](chapter2-3-auth-integrate-openldap-3.md)
4. [Replication 설정](chapter2-3-auth-integrate-openldap-4.md)
5. [Client 설정](chapter2-3-auth-integrate-openldap-5.md)
6. [LDAP Data 관리](chapter2-3-auth-integrate-openldap-6.md)
7. [Sudo LDAP 연동](chapter2-3-auth-integrate-openldap-7.md)