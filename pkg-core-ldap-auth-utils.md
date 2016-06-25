# ldap-auth-utils

### Descriptions:

LDAP 인증 통합 도구

### Features:
Support follow commands and config file:
* /etc/openldap/ldap-auth-utils.conf
* /usr/sbin/ldap_auth
* /usr/sbin/ldap_auth_init
* /usr/sbin/ldap_grpadd
* /usr/sbin/ldap_grpdel
* /usr/sbin/ldap_host_manage
* /usr/sbin/ldap_replica
* /usr/sbin/ldap_ssl
* /usr/sbin/ldap_useradd
* /usr/sbin/ldap_userdel

### Reference:
* https://github.com/Joungkyun/ldap-auth-utils
* [안녕 3 사용자 가이드: openldap 인증 통합](https://www.gitbook.com/book/joungkyun/annyung-3-user-guide/edit#/edit/master/chapter2-3-auth-intergrate-openldap.md)

### Dependencies:
* None

### Sub Packages:

* **ldap-auth-utils-passwd**  
  clent에서 사용자들이 암호를 변경하기 위한 도구

### Related Packages:

* [genpasswd](pkg-core-genpasswd.md)
* openldap-servers
* openldap-clients