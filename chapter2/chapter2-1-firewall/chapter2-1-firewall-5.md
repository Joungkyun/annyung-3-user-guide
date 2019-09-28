# User defined rule 제어

> 목차 1. 개요 2. Pre user defined rule 1. Syntax 2. Case study 3. Post user defined rule 1. Syntax 2. Case study

## 1. 개요

_**oops-firewall**_은 일종의 iptables rule을 생성하는 template 같은 도구 이기 때문에, rule을 만들다 보면 의도하는 바가 제한이 될 수 있습니다.

그렇게 때문에 사용자 rule을 등록 할 수 있도록 지원을 합니다.

이 기능은 ipatbles rule에 대해 이해를 하고 있어야 사용을 할 수 있으니 주의 하시기 바랍니다.

일단, iptables rule은 먼저 적용한 rule이 우선권을 가지는 순차 적용 방식의 특징을 가지고 있습니다.

```bash
  [root@an3 ~]$ iptables -A INPUT -j DROP
  [root@an3 ~]$ iptables -A INPUT -s 1.1.1.0 -j ACCEPT
```

상기 rule은 비록 1.1.1.0에서의 접근을 허가를 하고 있지만, 먼저 모든 INPUT packet을 drop 시키라고 하였기 때문에 결과적으로는 1.1.1.0에서의 접근이 불가 합니다. 그러므로 원하는 바를 실현하기 위해서는 다음과 같이 rule을 적용해야 합니다.

```bash
  [root@an3 ~]$ iptables -A INPUT -s 1.1.1.0 -j ACCEPT
  [root@an3 ~]$ iptables -A INPUT -j DROP
```

그러므로 _**oops-firewall**_에서 설정한 rule들이 어떤 순서로 적용이 되는지는 굉장히 중요한 부분이며, user defined rule 작성에도 많은 영향을 미칠 수 있습니다.

다음은 _**oops-firewall**_의 설정이 적용이 되는 순서를 보여 줍니다.

1. ALLOWSELF             \(filter.conf\)
2. ALLOWALL              \(filter.conf\)
3. BURTE\_FORCE\_FILTER    \(application.conf\)
4. pre user define rule  \(user.conf % directive\)
5. OUT\_TCP\_ALLOWPORT     \(filter.conf\)
6. OUT\_UDP\_ALLOWPORT     \(filter.conf\)
7. BR\_OUT\_TCP\_ALLOW      \(bridge.conf\)
8. BR\_OUT\_UDP\_ALLOW      \(bridge.conf\)
9. TCP\_ALLOWPORT         \(filter.conf\)
   1. TCP\_HOSTPERPORT       \(filter.conf\)
   2. UDP\_ALLOWPORT         \(filter.conf\)
   3. UDP\_HOSTPERPORT       \(filter.conf\)
   4. BR\_TCP\_ALLOW          \(bridge.conf\)
   5. BR\_UDP\_ALLOW          \(bridge.conf\)
   6. ICMP\_HOSTPERPING      \(filter.conf\)
   7. ICMP\_HOSTPERTRACE     \(filter.conf\)
   8. BR\_PING\_ALLOW         \(bridge.conf\)
   9. BR\_TRACE\_ALLOW        \(bridge.conf\)
   10. MASQ\_MATCH\_START      \(masq.conf\)
   11. masquerading rule     \(masq.conf\)
   12. ALL\_FORWARD\_TO        \(forward.conf\)
   13. TCP\_FORWARD\_TO\_SS     \(forward.conf\)
   14. TCP\_FORWARD\_TO\_S      \(forward.conf\)
   15. TCP\_FORWARD\_TO        \(forward.conf\)
   16. UDP\_FORWARD\_TO\_SS     \(forward.conf\)
   17. UDP\_FORWARD\_TO\_S      \(forward.conf\)
   18. UDP\_FORWARD\_TO        \(forward.conf\)
   19. post user define rule \(user.conf @ directive\)
   20. USE\_TOS               \(tos.conf\)

_**oops-firewall**_의 user defined rule은 _**/etc/oops-firewall/user.conf**_에서 적용을 합니다.

```bash
  [root@an3 ko]$ cat user.conf
  ##########################################################################
  # 사용자 정의
  # $Id: user.conf 338 2013-01-04 19:17:13Z oops $
  #
  # 이 파일의 설정은 OOPS Firewall 에서 지원하지 못하는 부분에 대해 사용자
  # 정의로 설정을 하는 파일이다.
  #
  # 이 파일을 설정함에있어 다음의 사항을 주의해야 한다. 이 파일에서는 OOPS
  # Firewall 이 실행하기 전에 실행할 사용자 명령과 OOPS Firewall 이 실행후
  # 에 실행될 사용자 명령을 구분하여 설정을 하도록 되어있다. 이는 IPTABLES
  # 가 먼저 설정한 것에 대해 우선 적용이 되기 때문이다.
  #
  # 이 파일에서는 제일 첫열에 % 로 시작하는 것과 @ 로 시작하는 라인만 정규
  # 설정으로 인식이 되어 진다.
  #
  # OOPS Firewall 이 실행되기 전에 실행이 되어야 하는 명령은 해당명령 앞에
  # % 로 시작을 해야 한다.
  #
  # 명령은 iptables 명령어만 빼고 옵션만 적어 주면 된다.
  #
  # 만약, 추가적인 netfilter 모듈을 사용하는 룰을 이용한다면, moduless.list
  # 에 해당 모듈 이름을 등록해 줘야 한다.
  #
  # 먼저 실행할 설정 (명령행 제일 앞에 % 가 있다.)
  #%-t mangle -A FORWARD -p tcp --sport 90 -j TOS --set-tos 0x10
  #
  # OOPS Firewall 이 실행된 후에 실행이 되어야 하는 명령을 경우에는 아래와
  # 같다.
  #
  # 나중 설정이 실행 (명령행 앞에 @ 이 있다)
  #@-t mangle -A FORWARD -p tcp --sport 94 -j TOS --set-tos 0x10
  [root@an3 ko]$
```

사용자 정의 rule은 위의 설명과 같이 _**pre rule**_과 _**post rule**_ 두가지로 구분이 됩니다.

_**pre rule**_ 이라는 것은 oops-firewall이 rule을 적용하기 전에 적용을 한다는 의미이고, _**post rule**_은 oops-firewall이 rule을 모두 적용한 후에 적용을 한다는 의미입니다.

보통은 _**pre rule**_을 많이 사용을 하며, _**post rule**_을 사용하는 경우는 oops-firewall이 적용한 rule 중간에 내가 원하는 rule을 끼워 넣고 싶을 경우 _**post rule**_을 사용하면 됩니다.

상단의 _**oops-firewall**_ 룰 적용 순서를 보면, 4번째가 _**pre rule**_이 적용이 되고, 28번째에 _**post rule**_이 적용이 되는 것을 확인 하실 수 있습니다.

## 2. Pre user defined rule

### 2.1. Syntax

user defined rule은 iptables 명령을 직접 실행하는 것과 거의 동일하게 설정이 됩니다. 예를 들어

```bash
  [root@an3 ~]$ iptables -A INPUT -s 10.0.0.1 -j ACCEPT
```

위와 같은 rule을 _**pre user defined rule**_ 로 설정을 하기 위해서, _**user.conf**_에 다음과 같이 설정을 합니다.

```bash
  %-A INPUT -s 10.0.0.1 -j ACCEPT
```

즉, shell에서 실행했던 commaind line중 실행 명령어\(여기서는 iptables\)를 **'%'**로 치환해 주면 됩니다. rule의 제일 처음이 **'%'**로 시작하면, _**pre rule**_로 처리가 되고, **'@'**으로 시작이 되면 _**post rule**_로 처리가 되는 것입니다.

### 2.2. Case study

_**pre rule**_을 자주 사용할 경우에 대해서 기술 합니다.

1. 원격에서 방화벽 작업을 할 경우 rule 을 잘못 작성할 경우, 작업자 마저 block 될 수 있습니다. 그러므로 _**pre rule**_에서 작업자의 IP를 미리 등록해 주고 작업을 합니다.  
  
    예를 들어, 작업자가 원격\(123.112.0.5\)에서 작업을 한다고 가정을 하면,

   ```bash
    %-A INPUT -s 123.112.0.5 -j ACCEPT
   ```

   위와 같은 작업을 해 둔다면, 방화벽 설정 중에 rule이 잘못되어 rule이 적용이 되다가 중지가 되어도 이에 영향을 받지 않고 123.112.0.5에서는 계속 작업이 가능하게 됩니다.

2. _**oops-firewall**_은 기본적으로 white list 정책이기 때문에 deny 정책이 없습니다. 그러므로 명확하게 deny 할 정책이 있다면, _**pre rule**_에서 처리할 수 있습니다.  
  
    예를 들어, 123.1.1.2에서 계속적인 spam 등록 시도가 있다면 다음과 같이 rule을 적용할 수 있습니다.

   ```bash
    %-A INPUT -s 123.1.1.2 -j DROP
   ```

   막는 경우에는 _**DROP**_ 또는 _**REJECT**_를 사용할 수 있습니다. _**DROP**_의 경우에는 말 그대로 오는 들어오는 packet에 대응을 하지 않기 때문에 상대방에서는 이 서버의 상태를 알 수가 없습니다. 하지만 _**REJECT**_를 사용할 경우에는 들어오는 packet에 대해서 reset을 보내기 때문에 상대 측에서 filtering을 하고 있다는 것을 인지할 수 있습니다. 이 차이점을 생각해서 _**DROP**_이나 _**REJECT**_를 사용하시면 됩니다.

3. 특정 포트가 anywhere로 허가된 상태에서 특정 곳에서의 해당 포트로 접근을 막고 싶을 경우 _**pre rule**_에서 처리할 수 있습니다. 이 경우는 2번째 경우와 동일하게 처리할 수 있습니다.

## 3. Post user defined rule

### 3.1. Syntax

_**post rule**_ 역시 _**pre rule**_의 문법과 동일하며, 처음의 _**'%'**_ 대신 _**'@'**_을 사용해 주면 됩니다. 예를 들어

```bash
  [root@an3 ~]$ iptables -A INPUT -s 10.0.0.1 -j DROP
```

위와 같은 rule을 _**post user defined rule**_ 로 설정을 하기 위해서, _**user.conf**_에 다음과 같이 설정을 합니다.

```bash
  @-A INPUT -s 10.0.0.1 -j DROP
```

즉, shell에서 실행했던 commaind line중 실행 명령어\(여기서는 iptables\)를 **'@'**으로 치환해 주면 됩니다.

### 3.2. Case study

_**post rule**_을 사용하는 경우는 _**oops-firewall**_이 생성한 rule 중간에 rule 을 끼워 놓고 싶을 경우 외에는 사용할 일이 거의 없습니다.

그러므로, 여기서는 rule을 끼워 넣는 방법에 대해서 기술을 합니다.

일단 INPUT chain에 rule을 끼워 넣어 보겠습니다.

1. 현재 INPUT chain의 rule을 line-number와 함께 확인 합니다.

   ```bash
    [root@an3 ~]$ iptables -L INPUT -n --line-number
    Chain INPUT (policy ACCEPT)
    num  target     prot opt source        destination
    1    DROP       all  --  0.0.0.0/0     0.0.0.0/0   state INVALID
    2    ACCEPT     all  --  0.0.0.0/0     0.0.0.0/0   state RELATED,ESTABLISHED
    3    ACCEPT     icmp --  0.0.0.0/0     0.0.0.0/0   icmptype 0
    4    ACCEPT     icmp --  0.0.0.0/0     0.0.0.0/0   icmptype 11
    5    ACCEPT     icmp --  0.0.0.0/0     0.0.0.0/0   icmptype 3 code 3
    6    ACCEPT     all  --  0.0.0.0/0     0.0.0.0/0
    7    ACCEPT     all  --  21.27.1.26 0.0.0.0/0
    8    ACCEPT     all  --  21.27.1.29 0.0.0.0/0
    9    ACCEPT     all  --  1.25.19.42  0.0.0.0/0
    10              tcp  --  0.0.0.0/0     0.0.0.0/0   tcp dpt:22 state NEW recent: SET name: OFIRE_22 side: source mask: 255.255.255.255
    11   DROP       tcp  --  0.0.0.0/0     0.0.0.0/0   tcp dpt:22 state NEW recent: UPDATE seconds: 60 hit_count: 10 TTL-Match name: OFIRE_22 side: source mask: 255.255.255.255
    12   ACCEPT     udp  --  0.0.0.0/0     0.0.0.0/0   udp dpts:33434:33525
    13   ACCEPT     udp  --  0.0.0.0/0     0.0.0.0/0   udp dpts:44444:44624
    14   DROP       tcp  --  0.0.0.0/0     0.0.0.0/0
    15   DROP       udp  --  0.0.0.0/0     0.0.0.0/0
    16   DROP       icmp --  0.0.0.0/0     0.0.0.0/0
    [root@an3 ~]$
   ```

2. 2번 rule 앞에 "10.0.0.1/32 에서의 접근 허가 rule"을 넣도록 합니다.

   ```bash
    [root@an3 ~]$ iptables -I INPUT 2 -s 10.0.0.1/32 -j ACCEPT
   ```

3. 2번째 rule로 들어갔는지 확인해 봅니다.

   ```bash
    [root@an3 ~]$ iptables -nL INPUT --line-number
    Chain INPUT (policy ACCEPT)
    num  target     prot opt source        destination
    1    DROP       all  --  0.0.0.0/0     0.0.0.0/0   state INVALID
    2    ACCEPT     all  --  10.0.0.1      0.0.0.0/0
    3    ACCEPT     all  --  0.0.0.0/0     0.0.0.0/0   state RELATED,ESTABLISHED
    4    ACCEPT     icmp --  0.0.0.0/0     0.0.0.0/0   icmptype 0
    5    ACCEPT     icmp --  0.0.0.0/0     0.0.0.0/0   icmptype 11
    6    ACCEPT     icmp --  0.0.0.0/0     0.0.0.0/0   icmptype 3 code 3
    7    ACCEPT     all  --  0.0.0.0/0     0.0.0.0/0
    8    ACCEPT     all  --  21.27.1.26 0.0.0.0/0
    9    ACCEPT     all  --  21.27.1.29 0.0.0.0/0
    10   ACCEPT     all  --  1.25.19.42  0.0.0.0/0
    11              tcp  --  0.0.0.0/0     0.0.0.0/0   tcp dpt:22 state NEW recent: SET name: OFIRE_22 side: source mask: 255.255.255.255
    12   DROP       tcp  --  0.0.0.0/0     0.0.0.0/0   tcp dpt:22 state NEW recent: UPDATE seconds: 60 hit_count: 10 TTL-Match name: OFIRE_22 side: source mask: 255.255.255.255
    13   ACCEPT     udp  --  0.0.0.0/0     0.0.0.0/0   udp dpts:33434:33525
    14   ACCEPT     udp  --  0.0.0.0/0     0.0.0.0/0   udp dpts:44444:44624
    15   DROP       tcp  --  0.0.0.0/0     0.0.0.0/0
    16   DROP       udp  --  0.0.0.0/0     0.0.0.0/0
    17   DROP       icmp --  0.0.0.0/0     0.0.0.0/0
    [root@an3 ~]$
   ```

   2번 라인에 10.0.0.1에 대한 ACCEPT가 들어가 있는 것이 확인이 됩니다. 그럼, _**user.conf**_에

   ```bash
   @-I INPUT 2 -s 10.0.0.1/32 -j ACCEPT
   ```

   와 같이 등록을 해 주면 됩니다.

