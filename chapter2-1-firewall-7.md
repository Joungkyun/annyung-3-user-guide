# Chapter 2. Access Control
## 1. 안녕 리눅스 방화벽 설정
### oops-firewall 실행 방법

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

  **_oops-firewall_** 구동 방법에 주의할 점이 있습니다. 일단, **_oops-firewlall_** 명령어에는 적용되어 있는 ruleset을 내릴 수 있는 방법이 없다 그렇기 때문에 rule set을 모두 내리기 위해서는 무조건 아래의 방법만이 가능합니다.
  
  ```bash
    [root@an3 ~]$ service oops-firewall stop
  ```

 그리고, oops-firewall을 구동하는 방법은 3번 항목에서 기술한 service 또는 systemctl 명령을 이용하는 방법과 여기서 기술할 **_oops-firewall_** 명령을 직접 실행하는 방법이 있습니다.
  
  이 둘의 차이를 잘 이해를 해야 하는데, 이는 설정 작업 환경에 따라 유리한 부분이 있기 때문입니다.
  
  먼저, 원격에서 작업을 한다면, **_oops-firewall_**을 직접 구동하는 것 보다는 **_serivce_**나 **_systemctl_**을 이용해서 구동하는 권장한다. 이유는, **_service_**나 **_systemctl_**을 이용하여 구동을 할 경우, rule set 적용에 에러가 발생할 경우 먼저 적용된 rule들을 **_rollback_**하는 기능이 있기 때문입니다.
  
  그리고, **_oops-firewall_**을 직접 구동하는 경우의 장점은 **_oops-firewall_**을 실행할 경우 기존의 rule set을 모두 초기화 한 후에 다시 적용하는 구조이기 때문에 굳이 restart를 할 필요가 없기 때문입니다.
  
  또한, -v 옵션을 주면 **_oops-firewall_**이 어떻게 ruleset을 적용하는지를 모두 출력해 주기 때문에 내가 만든 정책이 잘 반영이 되었는지 확인이 가능합니다.
  
  또한 -t 옵션을 같이 주면, 실제 deploy는 하지 않고 적용될 rule set을 미리 확인을 할 수도 있습니다.
  
  어떻게 구동을 하든지는 편리한 쪽을 선택하면 됩니다.
  
  다음은 **_oops-firewall_**에 -v 옵션을 주고 실행했을 때의 출력 메시지 입니다.

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

      * ETH0 정보
        IP 주소                                    : 12.1.87.15
        서브넷 마스크                              : 255.255.255.224
        네트워크                                   : 12.1.87.64
        네트워크 프리픽스                          : 27

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

      * 외부로의 traceroute 허가
        iptables -a input   -p icmp --icmp-type time-exceeded -j accept
        iptables -a input   -p icmp --icmp-type port-unreachable -j accept
        iptables -a output  -p udp --dport 33434:33525 -j accept
        iptables -a output  -p udp --dport 44444:44624 -j accept  

    10. 모든 서비스 허가

      * iptables -A INPUT  -i lo -j ACCEPT
      * iptables -A OUTPUT -o lo -j ACCEPT
      * iptables -A INPUT  -s 21.27.1.26 -j ACCEPT
      * iptables -A OUTPUT -d 21.27.1.26 -j ACCEPT
      * iptables -A INPUT  -s 21.27.1.29 -j ACCEPT
      * iptables -A OUTPUT -d 21.27.1.29 -j ACCEPT
      * iptables -A INPUT  -s 1.25.19.42 -j ACCEPT
      * iptables -A OUTPUT -d 1.25.19.42 -j ACCEPT

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
