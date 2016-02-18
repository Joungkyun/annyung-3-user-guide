# 1. 안녕 리눅스 방화벽 설정

안녕 리눅스는 CentOS/RHEL 7이 기본으로 제공하는 [firewalld](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)를 사용하지 않고, 안녕 1.x 부터 제공해 오던 **[oops-firewall](core-pkg-oops-firewall.md)**을 제공합니다.

**oops-firewall**이나 **firewalld**는 모두 iptables를 backend로 하는 즉, iptables rule을 대신 작성해 주는 utility라고 볼 수 있습니다. 즉, **oops-firewall**이나 **firewalld**에서 설정은 iptables rule을 작성한 것이고, 이 rule을 ipatbles로 deploy하여 netfilter에 반영을 하게 되는 것입니다.

한마디로, 어려운 iptables rule을 만들기 쉽게 도와주고, 정책을 생성/제거/반영을 쉽게 도와주는 도구라는 것이기 때문에 익숙한 것을 사용하시면 됩니다.

만약 CentOS/RHEL 7에서 제공하는 **firewalld**를 사용하기 원한다면, 다음의 절차를 따라 주십시오.

```sh
[root@an3 ~]$ yum remove oops-firewall
[root@an3 ~]$ yum install firewalld
[root@an3 ~]$ service firewalld enable
```

상기 작업을 완료 하였다면, **oops-firewall** 대신에 **firewalld**를 사용할 준비가 완료된 상태 입니다. 여기서 부터는 [RHEL 7 System Admin Guidel의 firewalld 부분]((https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html))을 참조 하십시오. 이 이하는 **oops-firewall**에 대한 기술을 진행 합니다.

이 문서에서는 **oops-firewall**에 대하여 바로 사용을 할 수 있는 대략적인 기법에 대해서만 기술을 합니다. **oops-firewall**의 전체적인 특징과 자세한 사용 방법은 [**oops-firewall** 사용 설명서](http://oops.org/?t=lecture&sb=firewall&n=2)를 참조 하십시오.

안녕 리눅스에서 **firewalld** 대신 **oops-firewall**을 제공하는 이유는 iptables rule에 대해서 전혀 지식이 없어서 사용을 할 수 있는 간단함과 명료한 설정이 하나이고, iptables를 잘 다를 수 있는 경우, **oops-firewall** 이 만든 rule의 쉽게 customizing을 할 수 있다는 점 입니다.


    ** 참고!

    다음 용어(표현)를 기억 하십시오.

    Inbound  - 외부(remote) 에서 내부(local)로 들어오는 접근
    Outbound - 내부(local) 에서 외부(remote)로 나가는 접근
    TCP/22   - TCP 22번 포트
    TCP/1.1.1.1:22 - TCP 1.1.1.1 IP의 22번 포트
    
    Rule 표현식:
    ------------
    
    * IP address 표현식
    
     - Anywhere(모든 IP) 표현식
       0.0.0.0/0 = 0/0 = ANYWHERE = anywhere
       
     - IP 범위 표현식
       1.1.1.1-255       => 1.1.1.1-1.1.1.255
       1.1.1.1-2.255     => 1.1.1.1-1.1.2.255
       1.1.1.1-2.3.255   => 1.1.1.1-1.2.3.255
       1.1.1.1-1.2.3.255 => 1.1.1.1-1.2.3.255
       
     - subnet
       10.10.10.0/24
       10.10.10.0/255.255.255.0

    * Port 표현식
      port[:state] => state가 지정되지 않으면 기본으로 NEW를 사용

      53
      53-100 (53번 부터 100번 포트까지의 범위 지정)
      53:NEW
      20:ESTABLISHED,RELATED

      => STATE
         NEW          : 새로운 연결
         ESTABLISHED  : 연결이 성립되어 있는 상태
         RELATED      : FTP의 20번 포트나 passive port와 같은 연관 연결
         INVALID      : 이상 패킷

    * 설정 표현식

      값의 구분자는 공백문자를 사용함.

        설정옵션 = 값1 값2 값3

      다음과 같이 값을 여러줄로 설정이 가능.
      마지막은 '\' 문자가 없어야 함. '\' 문자는 한줄로 연결한다는 의미를 뜻합니다.

        Directive = VALUE1 \
                    VALUE2 \
                    VALUE3



## 1.기본 설정

처음 안녕 리눅스를 설치를 했을 때 **oops-firewall**의 기본 설정 상태는 다음과 같습니다. 서비스 포트 번호에 대해서는 **_/etc/services_** 파일을 참조 하십시오.

  1. Inbound 허가
    * TCP/22 (**ssh**)
  2. Outbound 허가
    * TCP/21 (**FTP**)
    * TCP/22 (**SSH**)
    * TCP/25 (**SMTP**)
    * TCP/43 (**WHOIS**)
    * TCP/80 (**HTTP**)
    * TCP/443 (**HTTPS**)
    * TCP/873 (**RSYNC**)
    * UDP/53 (**DNS**)
    * UDP/123 (**NTP**)

상기와 같이 처음 설치 시에는 상당히 제약적으로 **ACL**이 설정이 되어 있습니다. 특히 안녕 리눅스 3에서 제공되는 **oops-firewall** 7 버전대는 outbound ACL 설정부분이 강화가 되어 이전 버전과는 달리 <u>명확하게 지정되지 않은 outbound 설정은 막히게 됩니다.</u>


## 2. **oops-firewall** 설정 파일

**oops-firewall**의 모든 설정 파일은 **_/etc/oops-firewall_**에 위치 합니다.

    * application.conf - layer 7 레벨의 제어
    * bridge.conf      - bridge packet 제어
    * filter.conf      - 기본적인 inbound/oubound packet 제어
    * forward.conf     - forwarding packet 제어
    * interface.conf   - Ethernet device 설정
    * masq.conf        - Masquerading 
    * modules.list     - Netfilter kernel module 제어
    * tos.conf         - TOS(Type of Service) 제어
    * user.conf        - 사용자 작성 rule
    
이 문서에서는 현재 까지는 기본적인 설정인 filter.conf와 user.conf 그리고 application.conf 의 일부에 대해서만 기술합니다. 향후, 더 많은 내용이 추가될 예정입니다. 이 외의 기능에 대해서는 [**oops-firewall** 사용 설명서](http://oops.org/?t=lecture&sb=firewall&n=2)를 참조 하십시오.

## 3. Inbound 제어

Inbound 제어를 한다는 것은 여러가지 의미가 있습니다.

    1. 외부의 사용자가 이 서버로 ssh 접속을 한다.
    2. 외부의 사용자가 이 서버로 http 접속을 한다.

이해하기 쉽게 표현하자면 위의 기술과 같이 외부의 사용자 또는 프로그램이 이 서버로 접근을 하는 것을 의미하며, 좀더 정확하게는 외부에서 들어오는 syn packet을 제어하는 것으로 이해를 하면 됩니다. (syn packet이 무엇인지 꼭!! 알야야 하는 것은 아닙니다)

**oops-firewall**의 Inbound 설정은 다음과 같습니다.

  ```bash
  [root@an3 ~]$ cat /etc/oops-firewall/filter.conf
  
  .. 상략..
  ##########################################################################
  # TCP configuration
  ##########################################################################
  #
  # 모두 열여줄 포트를 설정
  #
  # RULE:
  #       DESTINATION_PORT[:STATE]
  #
  TCP_ALLOWPORT = 22

  # 특정 호스트에 특정 포트를 열어 줄때
  #
  # RULE:
  #       SOURCE_IP[:DESTINATION_PORT[:STATE]]
  #
  TCP_HOSTPERPORT =

  ##########################################################################
  # UDP configuration
  ##########################################################################
  #
  # 모두 열여줄 포트를 설정
  # RULE:
  #       DESTINATION_PORT[:STATE]
  #
  UDP_ALLOWPORT =

  # 특정 호스트에 특정 포트를 열어 줄때
  #
  # RULE:
  #       SOURCE_IP[:DESTINATION_PORT[:STATE]]
  #
  UDP_HOSTPERPORT =

  ##########################################################################
  # ICMP configuration
  ##########################################################################
  #
  # 특정 호스트에 ping 을 열어 줄때
  # RULE:
  #       SOURCE_IP
  #
  ICMP_HOSTPERPING =

  # 특정 호스트에 traceroute 를 열여줄 때
  #
  # RULE:
  #       SOURCE_IP
  #
  ICMP_HOSTPERTRACE = anywhere
  .. 하략 ..
  
  [root@an3 ~]$
  ```
  
Inbound 설정은 filter.conf에서 하며, TCP/UDP/ICMP 이렇게 3개의 section으로 나뉘어져 있습니다. 각 섹션은 포트로만 제어를 하는 옵션과 호스트별 포트로 제어를 하는 경우로 나뉘어져 있습니다.
  
예를 들어 내 서버에서 web service를 하겠다고 한다면 아래와 같이 **TCP_ALLOWPORT**에 80번을 추가 합니다.
  
  ```ini
  TCP_ALLOWPORT = 22 **80**
  ```
  
하지만, 만약 모든 사람이 다 봐야 되는 경우가 아닐 수도 있습니다. 예를 들어 특정 업체와 연계를 하여, 그 업체의 특정 IP(또는 IP대역)에서 이 서버의 80번 포트로 접근을 허가해 주여야 하는 경우는 위의 설정 대신 아래와 같이**TCP_HOSTPERPORT** 에 설정을 해 줍니다.

  ```ini
  TCP_HOSTPERPORT = **10.0.48.2:80** **15.4.2.128/25:80**
  ```
  
위의 설정은 다음을 의미합니다.

  1. 10.0.0.48.2 에서 이 서버의 80번으로 들어오는 접근 허가
  2. 15.4.2.128-15.4.2.255 IP 대역에서 이 서버의 80번으로 들어오는 접근 허가

<u>주의 할 것은, IP는 **_외부에서 접근할 IP_**이고, 포트는 **_외부에서 접근할 이 서버의 포트_** 입니다.</u>

이런 형식으로 서비스에 맞게 protocol을 지정해 주실 수 있습니다.


## 3. Outbound 제어

**oops-firewall**의 기본 outbound rule은 왠만한 것은 동작하도록 기본 설정이 되어 있습니다만, 간혹 외부 서비스와의 연계 때문에 outbound 설정을 해야 하는 경우가 있습니다.

일단, outbound 설정은 inbound와 같이 **_/etc/oops-firewall/filter.conf_** 에서 합니다.


  ```bash
  [root@an3 ~]$ cat /etc/oops-firewall/filter.conf
  
  ** 상략 **
  
  ##########################################################################
  # 외부 서비스 이용을 위한 filtering 설정
  #

  ##########################################################################
  # TCP configuration
  ##########################################################################
  #
  # 내부에서 외부로의 접속을 허가할 TCP 포트 설정
  #
  # RULE:
  #       DESTINATION_PORT[:STATE]
  #
  OUT_TCP_ALLOWPORT = 21 22 25 43 80 123 443 873

  # 내부에서 접속할 외부 특정 호스트와 TCP 포트 설정
  #
  # RULE:
  #       DESTINATION_IP[:DESTINATION_PORT[:STATE]]
  #
  OUT_TCP_HOSTPERPORT =

  ##########################################################################
  # UDP configuration
  ##########################################################################
  #
  # 내부에서 외부로의 접속을 허가할 UDP 포트 설정
  #
  # RULE:
  #       DESTINATION_PORT[:STATE]
  #
  OUT_UDP_ALLOWPORT = 53 161

  # 내부에서 접속할 외부 특정 호스트와 UDP 포트 설정
  #
  # RULE:
  #       DESTINATION_IP[:DESTINATION_PORT[:STATE]]
  #
  OUT_UDP_HOSTPERPORT =
  ```

Inbound와 마찬가지로 TCP/UDP 섹션과 포트 설정, 호스트:포트 설정으로 나뉘어 있습니다.

이 설정을 변경해야 하는 경우는 다음과 같습니다.

결제 서비스와 연동을 해야 하는데, 결재사와의 통신을 15.2.0.1의 UDP 8000번 포트로 socket 통신을 해야 하는 경우, **oops-firewall**에서는 기본으로 8000번으로의 통신을 허가 하지 않기 때문에 다음과 같이 설정을 해 주어야 합니다.

  ```ini
  OUT_UDP_HOSTPERPORT = **15.2.0.1:8000**
  ```