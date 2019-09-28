# Outbound 제어

**oops-firewall**의 기본 outbound rule은 왠만한 것은 동작하도록 기본 설정이 되어 있습니다만, 간혹 외부 서비스와의 연계 때문에 outbound 설정을 해야 하는 경우가 있습니다.

일단, outbound 설정은 inbound와 같이 _**/etc/oops-firewall/filter.conf**_ 에서 합니다.

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

```bash
  OUT_UDP_HOSTPERPORT = 15.2.0.1:8000
```

ICMP의 경우는 outbound에 대해서는 모두 열려 있으며, ICMP outbound 제어는 user.conf에서 사용자 설정으로 제어해야 합니다.

