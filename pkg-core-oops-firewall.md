# oops-firewall

### Description:
NETFILTER 기능을 이용하여 원격 공격으로 부터 시스템을 안전하게
막기 위하여 사용을 할수있다.

oops-firewall 은 IP/포트 필터링, 매스커래이딩, 브릿지 네트워크를 쉽게 설정하도록 도와준다.

### Features:
* 설정 파일
 * _/etc/oops-firewall/application.conf_ - brute force 방어 및 iptables layer7 extension 설정
 * _/etc/oops-firewall/bridege.conf_ - bridge device filter
 * _/etc/oops-firewall/filter.conf_ - main filter
 * _/etc/oops-firewall/forward.conf_ - forwarding filter
 * _/etc/oops-firewall/interface.conf_ - interface configration
 * _/etc/oops-firewall/masq.conf_ - masquerading filter
 * _/etc/oops-firewall/tos.conf_ - TOS configration
 * _/etc/oops-firewall/user.conf_ - User defined filter

### Reference:
* [안녕리눅스 방화벽 설정](chapter2-1-firewall.md)
* [oops-firewall 사용 설명서](http://oops.org/?t=lecture&sb=firewall&n=2)
* http://svn.oops.org/wsvn/OOPS.oops-firewall/trunk/doc/ko/


### Dependencies:
* iptables

### Sub Packages:
* None

### Releated Packages:
* [firewalld](https://access.redhat.com/documentation/ko-KR/Red_Hat_Enterprise_Linux/7/html/Migration_Planning_Guide/ch04s11.html)
* iptables-init
