# Chapter 2. Access Control
## 1. 안녕 리눅스 방화벽 설정
### 7. oops-firewall 실행 방법

1. booting 시 oops-firewall 이 구동 되도록 설정
  ```bash
  [root@an3 ~]$ /sbin/service oops-firewall enable
  [root@an3 ~]$ systemctl enable oops-firewall
  ```
2. booting 시 oops-firewall 이 구동되지 않도록 설정
  ```bash
  [root@an3 ~]$ /sbin/service oops-firewall disable
  [root@an3 ~]$ systemctl disable oops-firewall
  ```
3. init(or systemd)를 이용한 oops-firewall 구동
  ```bash
  [root@an3 ~]$ /sbin/service oops-firewall [start|stop|restart|status]
  [root@an3 ~]$ systemctl [start|stop|restart|status] oops-firewall
  ```
4. 실행 명령을 이용한 oops-firewall 구동

```bash
  [root@an3 ~]$ oops-firewall -h

  ############################################################################
  # OOPS Firewall - 설정이 간단한 iptables 프론트엔드 v7.0.2
  ############################################################################

  사용법: oops-firewall -[옵션]
  옵션:
           -c config_directory    설정 파일 디렉토리 지정
           -t                     테스트 모드로 실행
           -v                     상세 모드
           -n                     안시 모드 출력 하지 않음
           -V                     현재 버전 출력
           -h                     도움말

  [root@an3 ~]$ oops-firewall -t -v

  ############################################################################
  # OOPS Firewall - 설정이 간단한 iptables 프론트엔드 v7.0.2
  ############################################################################


  1. 실행 유저 체크                              : OK

  2. 커널 버전 검사                              : OK

  3. IP 주소 체크

  4. 인터페이스 정보

    * FIREWALL_WAN                               : eth0
    * MASQUERADE_WAN                             : Not Set
    * MASQUERADE_LOC                             : Not Set

  5. 넷필터 초기화

    * 넷필터 모듈 등록
      ip_conntrack 모듈 등록                     : 존재함
      ip_tables 모듈 등록                        : 존재함
      ip_conntrack_ftp 모듈 등록                 : 존재함

    * 기본 테이블 초기화
      사슬 카운터 초기화                         : OK
      모든 INPUT 사슬 허가                       : OK
      INPUT 사슬의 규칙 제거                     : OK
      모든 OUTPUT 사슬을 허가                    : OK
      FORWARD 사슬의 규칙 제거                   : OK
      모든 FORWARD 사슬 허가                     : OK
      OUTPUT 사슬의 규칙 제거                    : OK

    * NAT 테이블 초기화
      NAT 테이블 카운터 초기화                   : OK
      FORWARD 규칙 제거                          : OK
      PREROUTING 규칙 제거                       : OK
      모든 POSTROUTING 사슬 허가                 : OK
      POSTROUTING 규칙 제거                      : OK

    * Mangle 테이블 초기화
      MANGLE 테이블 카운터 초기화                : OK
      FORWARD 규칙 제거                          : OK
      PREROUTING 규칙 제거                       : OK
      모든 OUTPUT 사슬을 허가                    : OK
      OUTPUT 규칙을 제거                         : OK
      모든 INPUT 사슬 허가                       : OK
      INPUT 규칙을 제거                          : OK
      모든 FORWARD 사슬 허가                     : OK
      FORWARD 규칙 제거                          : OK
      모든 POSTROUTING 사슬 허가                 : OK
      POSTROUTING 규칙 제거                      : OK

  6. Bridge 모드 초기화

    * BRIDGE 모드가 셋팅되지 않았습니다.

  7. 기본 커널 파라미터 설정

    * TCP SYNCOOKIES 파라미터 사용               : OK
    * all IP 스푸핑 체크                         : OK
    * default IP 스푸핑 체크                     : OK
    * eth0 IP 스푸핑 체크                        : OK
    * eth1 IP 스푸핑 체크                        : OK
    * eth2 IP 스푸핑 체크                        : OK
    * eth3 IP 스푸핑 체크                        : OK
    * lo IP 스푸핑 체크                          : OK
    * 유효하지 않은 메세지 거절                  : OK
    * BroadCast 로의 Ping 을 거부                : OK

  8. 이상 패킷 제거

    * iptables -A INPUT   -m state --state INVALID -j DROP
    * iptables -A OUTPUT  -m state --state INVALID -j DROP

  9. 기본 규칙 허가

    * 모든 ESTABLISED/REALTED 연결 허가
      iptables -A INPUT   -m state --state ESTABLISHED,RELATED -j ACCEPT
      iptables -A OUTPUT  -m state --state ESTABLISHED,RELATED -j ACCEPT

    * 외부로의 PING 허가
      iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPT
      iptables -A INPUT  -p icmp --icmp-type echo-reply -j ACCEPT

    * 외부로의 Traceroute 허가
      iptables -A INPUT   -p icmp --icmp-type time-exceeded -j ACCEPT
      iptables -A INPUT   -p icmp --icmp-type port-unreachable -j ACCEPT
      iptables -A OUTPUT  -p udp --dport 33434:33525 -j ACCEPT
      iptables -A OUTPUT  -p udp --dport 44444:44624 -j ACCEPT

  10. 모든 서비스 허가

    * iptables -A INPUT  -i lo -j ACCEPT
    * iptables -A OUTPUT -o lo -j ACCEPT
    * iptables -A INPUT  -s 211.237.1.226 -j ACCEPT
    * iptables -A OUTPUT -d 211.237.1.226 -j ACCEPT
    * iptables -A INPUT  -s 211.237.1.229 -j ACCEPT
    * iptables -A OUTPUT -d 211.237.1.229 -j ACCEPT
    * iptables -A INPUT  -s 1.235.149.42 -j ACCEPT
    * iptables -A OUTPUT -d 1.235.149.42 -j ACCEPT

  11. 전처리 사용자 설정 규칙 추가

    * Brute Force 공격 필터
      iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
               -m recent --set --name OFIRE_22
      iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
               -m recent --update --seconds 60 --hitcount 10 --rttl \
               --name OFIRE_22 -j DROP

    * 실행 전 유저 명령(%) 이 설정되어 있지 않음

  12. 외부로 나가는 서비스 허가

    * TCP 서비스
      iptables -A OUTPUT  -p tcp  --dport 21    -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p tcp  --dport 22    -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p tcp  --dport 25    -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p tcp  --dport 43    -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p tcp  --dport 80    -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p tcp  --dport 443   -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p tcp  --dport 873   -m state --state NEW -j ACCEPT

    * TCP per HOST 서비스

    * UDP 서비스
      iptables -A OUTPUT  -p udp  --dport 53    -m state --state NEW -j ACCEPT
      iptables -A OUTPUT  -p udp  --dport 123   -m state --state NEW -j ACCEPT

    * UDP per HOST 서비스


  13. 내부로 들어오는 서비스 허가

    * TCP 서비스

    * TCP per HOST 서비스

    * UDP 서비스

    * UDP per HOST 서비스

    * ICMP 서비스 설정
      ==> ping 서비스 설정

      ==> traceroute 서비스 설정
        iptables -A OUTPUT  -s 0.0.0.0/0       -p icmp --icmp-type time-exceeded    -j ACCEPT
        iptables -A OUTPUT  -s 0.0.0.0/0       -p icmp --icmp-type port-unreachable -j ACCEPT
        iptables -A INPUT   -s 0.0.0.0/0       -p udp  --dport 33434:33525 -j ACCEPT
        iptables -A INPUT   -s 0.0.0.0/0       -p udp  --dport 44444:44624 -j ACCEPT


  14. 매스쿼레이드 설정


  15. 포워딩 설정

    * 만료된 지시자 검사


  16. 후처리 사용자 설정 규칙 추가

    * 실행 후 유저 명령(@)이 설정되어 있지 않음

  17. TOS 설정

    * INPUT 체인 Tos 설정
      iptables -t mangle -A INPUT -p tcp --dport 21 -j TOS --set-tos 0x08
      iptables -t mangle -A INPUT -p tcp --sport 21 -j TOS --set-tos 0x08
      iptables -t mangle -A INPUT -p tcp --dport 22 -j TOS --set-tos 0x10
      iptables -t mangle -A INPUT -p tcp --sport 22 -j TOS --set-tos 0x10
      iptables -t mangle -A INPUT -p tcp --dport 23 -j TOS --set-tos 0x10
      iptables -t mangle -A INPUT -p tcp --sport 23 -j TOS --set-tos 0x10
      iptables -t mangle -A INPUT -p tcp --dport 80 -j TOS --set-tos 0x04
      iptables -t mangle -A INPUT -p tcp --sport 80 -j TOS --set-tos 0x04
      iptables -t mangle -A INPUT -p tcp --dport 443 -j TOS --set-tos 0x04
      iptables -t mangle -A INPUT -p tcp --sport 443 -j TOS --set-tos 0x04
      iptables -t mangle -A INPUT -p tcp --dport 1024:65535 -j TOS --set-tos 0x00
      iptables -t mangle -A INPUT -p tcp --sport 1024:65535 -j TOS --set-tos 0x00

    * OUTPUT 체인 Tos 설정
      iptables -t mangle -A OUTPUT -p tcp --dport 21 -j TOS --set-tos 0x08
      iptables -t mangle -A OUTPUT -p tcp --sport 21 -j TOS --set-tos 0x08
      iptables -t mangle -A OUTPUT -p tcp --dport 22 -j TOS --set-tos 0x10
      iptables -t mangle -A OUTPUT -p tcp --sport 22 -j TOS --set-tos 0x10
      iptables -t mangle -A OUTPUT -p tcp --dport 23 -j TOS --set-tos 0x10
      iptables -t mangle -A OUTPUT -p tcp --sport 23 -j TOS --set-tos 0x10
      iptables -t mangle -A OUTPUT -p tcp --dport 80 -j TOS --set-tos 0x04
      iptables -t mangle -A OUTPUT -p tcp --sport 80 -j TOS --set-tos 0x04
      iptables -t mangle -A OUTPUT -p tcp --dport 443 -j TOS --set-tos 0x04
      iptables -t mangle -A OUTPUT -p tcp --sport 443 -j TOS --set-tos 0x04
      iptables -t mangle -A OUTPUT -p tcp --dport 1024:65535 -j TOS --set-tos 0x00
      iptables -t mangle -A OUTPUT -p tcp --sport 1024:65535 -j TOS --set-tos 0x00

  18. 거부 규칙 설정


    * 모든 TCP 패킷을 Drop
      iptables -A INPUT   -p tcp  -j DROP
      iptables -A OUTPUT  -p tcp  -j DROP

    * 모든 UDP 패킷을 Drop
      iptables -A INPUT   -p udp  -j DROP
      iptables -A OUTPUT  -p udp  -j DROP

    * 모든 ICMP 패킷을 거부
      iptables -A INPUT   -p icmp -j DROP
      iptables -A OUTPUT  -p icmp -j DROP

  [root@an3 ~]$  
```
