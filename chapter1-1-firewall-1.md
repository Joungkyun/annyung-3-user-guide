# 1. 기본 설정

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
