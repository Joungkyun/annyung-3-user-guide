# LDAP data 관리

console에서의 LDAP data관리는 너무나도 불편합니다. 특히나 설정 하나 변경을 하려면 ldif 파일을 만들어서 ldapadd 또는 ldapmodify 명령을 실행을 해야 한다는 것은 정말 짜증 나는 일입니다.

그렇기 때문에, Gui Tool들을 많이 사용하게 됩니다.

보통 openldap 관련 문서들을 보면 대부분 LDAP gui tool로서 Web base의 [phpldapadmin](http://phpldapadmin.sourceforge.net/wiki/index.php/Main_Page)을 소개하고 있으며, 그 외에서 디자인적으로도 화려한 java 기반의 gui tool들이 많이 있습니다.

하지만, 필자가 인증 통합을 목적으로 ldap을 구축하였을 때, 가장 편하게 사용을 할 수 있었던 tool은 [LdapAdmin](http://www.ldapadmin.org/)이라는 Gui tool을 권장 합니다. 다만 windows용 밖에 지원을 하지 않기 때문에 다른 OS에서는 아직 제가 테스트를 해 본 적이 없어 검색을 해 보셔서 마땅한 tool을 선택 하시기 바랍니다.