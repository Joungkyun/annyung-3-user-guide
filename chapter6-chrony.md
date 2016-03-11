# Chrony

> **목차**
> 1. 개요
> 1. Chrony를 이용한 시간 동기화
> 1. Time server 구성


#1. 개요

***chrony***는 RHEL 7에서 기본으로 제공하는 NTP daemon/client 입니다. RHEL 7 부터 NTP 대신에 chrony를 제공하고 있으며, 안녕 리눅스 3에서도 ***chrony***를 기본 NTP daemon/client로 제공하고 있습니다.

물론, 기존의 NTP도 제공하고 있으므로, ***chrony***에 익숙하지 않아서 NTP를 사용하고 싶다면, ***chrony***를 제고하고 NTP를 설치할 수 있습니다. 다만, ***NTP***보다 ***chorny***가 설정이 더 간결하고 ***NTP***의 단점을 개선하고자 시작된 project 이기 때문에 ***chrony*** 사용을 권장 합니다. (물론 protocol이 호환이기 때문에 혼합해서 사용이 가능 합니다.)

이 문서에서는 ***chrony*** daemon을 이용하여 서버의 시간을 동기화 하는 방법과, ***chrony*** daemon을 이용하여 Time service를 하는 방법에 대해서만 기술하며, ***chronyc***를 이용하여 ***chronyd***를 관리하는 것에 대해서는 다루지 않습니다. ***chronyc***를 이용한 ***chrony*** daemon 관리에 대해서는 Redhat Enterprise Linux 7 [System Administrator Guide 15.3 Using Chrony](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Using_chrony.html)를 참고 하십시오.

Time server 설정 전에 우선 알아야 할 것이 ***Stratum*** 이라는 의미입니다. ***Stratum***은 지층이라는 의미로, NTP protocol은 피라미드 형식의 구성으로 이루어져 있기 때문에, ***Stratum*** 0은 피라미드의 꼭대기라고 비유할 수 있습니다.

***Stratum*** 0은 ***primary reference clock*** 이라고 부르며, NTP protocol과는 상관이 없습니다. 즉 직접적으로 시간 서비스를 하는 것은 아니며, ***Stratum*** 1로 시간을 전송하는 장비들을 말하며 ***primary reference clock*** 장비에는 *GPS*, *세슘 원자 시계* 등이 있습니다.

보통 ***Stratum*** 1 level의 서버들은 ***primary reference clock***에서 시간을 동기화 하여 서비스를 하며, NTP에서 최상위층이라고 생각하면 됩니다.

다만, ***Stratum*** 1 level의 서버들은 client들이 ***Stratum*** 1 서버에서 동기화를 하면 시간이 더욱 정확할 것이라는 생각으로 ***Stratum*** 1 서버들을 설정하여 서비스 부하가 높아져서 ***Stratum*** 2 서버들에만 접근을 허가하고 access를 막아 놓은 경우가 대부분 입니다.

또한, NTP 구성 목적이 대부분 정확한 시간 보다는 시간의 동기화에 있기 때문에 꼭 최상위 ***stratum***에 동기화를 할 이유가 별로 없기 때문에 ***Stratum*** 2 정도에 sync를 하는 것을 권장 합니다.

NTP protocol과 서비스에 대한 자세한 설명은 http://time.ewha.or.kr/ 을 참조 하십시오.

##2. Chrony를 이용한 시간 동기화


***chrony***는 NTP와 동일하게 Time service를 하지 않고 서버의 시간 동기화를 하기만 하려고 해도 기본적으로 daemon으로 구동을 해야 합니다.

안녕 리눅스 3은 설치시에 기본적으로 *Stratum 2* 또는 *Stratum 3*으로 구성된 *CentOS NTP pool*에 등록되어 있는 외부 Time server로 부터 시간을 동기화 하도록 되어 있습니다. 즉, 단순히 서버의 시간 동기화를 위해서라면, 별도의 설정을 건드릴 필요 없이 ***chronyd***만 실행을 시켜 주면 된다는 의미입니다.

만약, ***chrony***가 설치 되어 있지 않은 상태에서, 설치 후 시간 동기화를 하고 싶다면 다음 명령을 이용할 수 있습니다.

```bash
[root@an3 ~]$ yum install chrony     # ntp package가 설치 되어 있다면 삭제하고 설치해야 함
[root@an3 ~]$ service chronyd enable # booting 시에 구동
[root@an3 ~]$ service chronyd restart  # chronyd 시작
```

또한, 기본으로 설정이 되어 있는 *CentOS NTP pool*의 time server가 아니라 다른 Time server에서 동기화를 하고 싶다면 ***/etc/chrony/chrony.conf*** 에서 ***server*** 지시자에 원하는 Time server를 등록하면 됩니다.

```nginx
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```

만약 daemon으로 구동을 하기 싫다면, ***chronyd*** process를 중지시키고, ***ntpdate***를 이용하여 manual로 동기화 할 수 있습니다.

```bash
[root@an3 ~]$ service chronyd stop
[root@an3 ~]$ service chronyd disable
[root@an3 ~]$ yum install ntpdate
[root@ane ~]$ service ntpdate enable
```

위와 같이 명령을 실행을 하면, chronyd는 구동하지 않으며, booting 시에 ***ntpdate***로 시간 동기화를 하게 됩니다. 주의할 점은 부팅시에 ntpdate는 /etc/chrony/chrony.conf 또는 /etc/ntp/ntp.conf의 ***server***로 등록되어진 time server로 부터 동기화를 하기 때문에 chonry.conf 또는 ntp.conf에 time server가 설정 되어 있어야 합니다. (chrony.conf가 ntp.conf보다 우선 합니다.)


또한, 원할 때 ```ntpdate``` 명령을 실행함으로서 동기화를 할 수 있습니다.

```bash
[root@an3 ~]$ ntpdate 0.centos.pool.ntp.org
```

참고로, ntpd와는 달리 chronyd가 구동중이라도 ***ntpdate*** 실행이 가능 합니다.


##3. Time server 구성

많은 수의 서버를 관리할 경우, 모든 서버를 외부의 Time server를 이용하거나 또는 *CentOS NTP Pool*의 Time server를 이요할 경우, 서버마다 시간 차가 발생할 수 있습니다.

이럴 경우에는 local network에 Time server를 구성을 하고, 서버들이 이 Time server를 바라보게 하여 운영을 하는 것이 훨씬 더 좋습니다.

구성도를 보자면 다음과 같은 구성을 하는 것이 일반적 입니다.

![로컬 Time server 구성](stratum.jpg)


내부에 fail over를 위해 Stratum 3 level의 Time server 2대를 구성하고, private network 안에 있는 server/client 들이 이 time server를 통해서 시간 동기화를 하도록 구성을 하도록 합니다.

그리고, 2대의 Time server 간에는 peer 구성을 하여 서로 동기화를 하게 할 수 있지만, 제 개인적인 견해로는 Time service 특성상 peer 구성 보다는 그냥 master 2대로 구성하는 것이 관리상 더 편했던 것 같습니다. 그래서 여기서는 peer 구성은 하지 않고 그냥 time server 2대를 독립적으로 구성하되, sync할 stratum 2 level의 서버를 동일하게 지정하여 peer 설정을 한 것과 비슷하게 구성을 할 것입니다.

여기서는 다음의 환경으로 Time Server 1과 Time Server 2를 구성하는 예로 설명을 합니다.

* local network : 10.0.0.0/8
* Time Server 1 IP: 10.10.0.1
* Time Server 2 IP: 10.10.0.2
* Time Server 1과 2는 Stratum 3로 구성

Time server를 운영하는 데 있어 사용되는 resource는 극히 적습니다. 실제로 Stratum 2 서버들 중에는 현재 486 machine으로 동작하는 경우도 있습니다. 그러므로, DNS나 DHCP 서버에 Time server 설정을 하는 것을 권장 합니다.

###3.1 /etc/chrony/chrony.conf

상단의 구성 처럼 2대의 Time server를 구성할 경우, 두대의 Time server간의 동기화를 위해서 peer 설정을 하게 됩니다. 하지만, Time server의 source stratum이 동일할 경우 굳이 peer 설정이 의미가 별로 없기 때문에 그냥 독립적인 time server 2대를 구성하는 것도 문제가 없습니다.

일단 안녕 리눅스 3의 chrony.conf에 등록이 되어 있는 Time server들은 대부분 Stratum 2 level 입니다. 그러므로 굳이 이를 변경할 필요는 없습니다. 굳이 원하는 time server가 있다면 찾아서 변경해 주시면 됩니다.

http://time.ewha.or.kr/domestic.html 를 참고 하여 Stratum 2 3개 정도, Stratum 3 1개 정도를 선택 합니다.

***chrony***는 ***NTP***와는 달리 기본 설정에서 *access* 설정을 하지 않으면 UDP 123번 port를 listen 하지 않습니다. 즉, client mode로만 동작을 한다는 의미입니다. 그러므로 ***chrony.conf***에 ***allow*** 설정을 해 주어야 Time service가 가능합니다.

그리고 ***local*** 지시자로 이 서버가 Stratum 3임을 announce 하도록 설정 합니다.

```nginx
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# permit access from 10.0.0.0/8 network
allow 10.0.0.0/8

# announce this server as Stratum 3
#local stratum 10
local stratum 3
```

위와 같이 설정이 되어 있으면 됩니다. (기본 설정 파일에서 ***allow***와 ***local***만 추가가 됩니다.)

###3.2 방화벽 설정

방화벽이나 subnet 구간에 switch ACL이 있다면 time server 1(10.10.0.1)과 time server 2(10.10.0.2)의 UDP 123번 포트를 open 해 주어야 합니다.

다음은 time server 자체에서 방화벽(oops-firewall 또는 firewalld)이 운영중일 경우 입니다. private network이라서 방화벽을 구동하고 있지 않다면 무시 하십시오.

time server에 ***oops-firewall***이 실행 되고 있다면 ***/etc/oops-firewall/filter.conf***의 ***UDP_ALLOWPORT***에 *123*을 추가해 주십시오.

```bash
[root@an3 ~]$ cat /etc/oops-firewall/filter.conf
  ** 상략 **
###########################################################################
# UDP configuration
###########################################################################
#
# Port configuration to open for all connections
#
# RULE:
#       DESTINATION_PORT[:STATE]
#
UDP_ALLOWPORT = 123
  ** 하략 **
[root@an3 ~]$ oops-frewall -v  # oops-firewall 재구동
[root@an3 ~]$
```

time server에 ***firewalld***가 실행이 되고 있다면 다음과 같이 하십시오.

```bash
[root@an3 ~]$ firewall-cmd --permanent --zone=public --add-port=123/udp
```


###3.3 daemon 구동 및 서비스 확인

```bash
[root@an3 ~]$ service chronyd restart
[root@an3 ~]$ netstat -anp | grep ":123"
udp    0  0 0.0.0.0:123       0.0.0.0:*          16137/chronyd
[root@an3 ~]$ chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^- mail.funix.net                3   6    77    12  +6281us[+6281us] +/-  156ms
^+ dadns.cdnetworks.co.kr        2   6    77    12  -2475us[-2475us] +/-   61ms
^- send.mx.cdnetworks.com        2   6    77    13  -6073us[-6073us] +/-  113ms
^* 114.207.245.166               2   6    77    14  +1656us[+1597us] +/-   41ms
```



###3.4 client 설정
####3.4.1 /etc/chrony/chrony.conf

client 설정에서는 ***server*** 지시자만 새로 만든 time server 1과 time server 2를 지정해 주면 되며, 그 외에는 수정할 필요가 없습니다.

```nginx
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 10.10.0.1 iburst
server 10.10.0.2 iburst
```

####3.4.2 방화벽 설정

client에 ***oops-firewall***이 실행되고 있다면 ***OUT_UDP_HOSTPERPORT***에 time server의 123번 포트를 설정해 주십시오. private network이라서 ***oops-firewall***을 구동하고 있지 않다면 무시 하십시오.

```bash
[root@an3 ~]$ cat /etc/oops-firewall/filter.conf
  ** 상략 **
##########################################################################
# UDP configuration
##########################################################################
#
# Configuration of the Port
# Designate the ports of the services from teh internal point to the
# external point.
#
# RULE:
#       DESTINATION_PORT[:STATE]
#
OUT_UDP_ALLOWPORT = 53

# To open specific port on specific host
#
# RULE:
#       DESTINATION_IP[:DESTINATION_PORT[:STATE]]
#
OUT_UDP_HOSTPERPORT = 10.10.0.1:123 10.10.0.2:123

[root@an3 ~]$ oops-firewall -v  # oops-firewall 재구동
[root@an3 ~]$


####3.4.3 chrony 재시작 및 확인

***chrony***를 재시작 한 후, ***chronyc***를 이용하여 확인을 합니다.

```bash
[root@an3 ~]$ service chronyd restart
[root@an3 ~]$ chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 10.0.0.1                      3   6    77    14  +1656us[+1597us] +/-   3ms
^+ 10.10.0.2                     2   6    77    12  -2475us[-2475us] +/-   3ms
```