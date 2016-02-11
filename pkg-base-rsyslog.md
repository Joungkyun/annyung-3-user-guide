# rsyslog

### Description:
Enhanced system logging and kernel message trapping daemon

### Changes on AnNyung:
1. kernel message를 _/dev/console_ 에서 _/var/log/kernel_ 로 변경
2. mysql plugin에서 mysql unix socket을 사용할 수 있도록 수정
 * host이름이 **"/"**로 시작할 경우 unix domain socket으로 간주

### Sub packages:
* **rsyslog-crypto** - Encryption support
* **rsyslog-doc** - HTML Documentation for rsyslog
* **rsyslog-elasticsearch** - ElasticSearch output module for rsyslog
* **rsyslog-gnutls** - TLS protocol support for rsyslog
* **rsyslog-gssapi** - GSSAPI authentication and encryption support for rsyslog
* **rsyslog-libdbi** - Libdbi database support for rsyslog
* **rsyslog-mmaudit** - Message modification module supporting Linux audit format
* **rsyslog-mmjsonparse** - JSON enhanced logging support
* **rsyslog-mmnormalize** - Log normalization support for rsyslog
* **rsyslog-mmsnmptrapd** - Message modification module for snmptrapd generated messages
* **rsyslog-mysql** - MySQL support for rsyslog
* **rsyslog-pgsql** - PostgresSQL support for rsyslog
* **rsyslog-relp** - RELP protocol support for rsyslog
* **rsyslog-snmp** - SNMP protocol support for rsyslog
* **rsyslog-udpspoof** - Provides the omudpspoof module