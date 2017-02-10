# Chapter 5.5 DNSSEC 설정

참조: https://sites.google.com/site/dnsportalkorea/home
      http://krnic.or.kr/jsp/resources/dns/dnssecInfo/dnssecBind.jsp
      http://sangchul.kr/89
      
      
* KSK (Key signing key 생성)
  * ***ZSK***를 서명하는데 사용한다.
  
  ```bash
  [root@host dnssec-key]$ dnssec-keygen -3 -r /dev/urandom -b 2048 -n ZONE -f KSK joungkyun.org.
  Generating key pair.......................................+++ ....................................................................+++
  Kjoungkyun.org.+005+37828
 
  [root@host dnssec-key]$ ls -l
  -rw-r--r-- 1 root   root    605  2월 11 04:39 Kjoungkyun.org.+005+37828.key
  -rw------- 1 root   root   1774  2월 11 04:39 Kjoungkyun.org.+005+37828.private
  ```
* ZSK (Zone signing key 생성)
  * zone의 모든 ***RR***을 서명한다.
  
  ```bash
  [root@host ~]$ dnssec-keygen -3 -r /dev/urandom -b 2048 -n ZONE -I 20181124171134 -D 20181224171134 joungkyun.org.
  ```
* zone file sign
  * zone file에 key file 을 include
  
  ```bash
  [root@host ~]$ dnssec-signzone -S -K /var/named/etc/pki/dnssec-keys -3 291e6a -e 20181124171134 -o joungkyun.org. joungkyun.zone
  ```