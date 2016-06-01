# LDAP data 관리

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
