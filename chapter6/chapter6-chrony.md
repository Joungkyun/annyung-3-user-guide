# Chrony

## Chrony

> **목차**  
> 1. 개요  
> 2. Chrony를 이용한 시간 동기화  
> 3. Time server 구성 4. 윤초 대응

## 1. 개요

_**chrony**_는 RHEL 7에서 기본으로 제공하는 NTP daemon/client 입니다. RHEL 7 부터 NTP 대신에 chrony를 제공하고 있으며, 안녕 리눅스 3에서도 _**chrony**_를 기본 NTP daemon/client로 제공하고 있습니다.

물론, 기존의 NTP도 제공하고 있으므로, _**chrony**_에 익숙하지 않아서 NTP를 사용하고 싶다면, _**chrony**_를 제고하고 NTP를 설치할 수 있습니다. 다만, _**NTP**_보다 _**chorny**_가 설정이 더 간결하고 _**NTP**_의 단점을 개선하고자 시작된 project 이기 때문에 _**chrony**_ 사용을 권장 합니다. \(물론 protocol이 호환이기 때문에 혼합해서 사용이 가능 합니다.\)

이 문서에서는 _**chrony**_ daemon을 이용하여 서버의 시간을 동기화 하는 방법과, _**chrony**_ daemon을 이용하여 Time service를 하는 방법에 대해서만 기술하며, _**chronyc**_를 이용하여 _**chronyd**_를 관리하는 것에 대해서는 다루지 않습니다. _**chronyc**_를 이용한 _**chrony**_ daemon 관리에 대해서는 Redhat Enterprise Linux 7 [System Administrator Guide 15.3 Using Chrony](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Using_chrony.html)를 참고 하십시오.

Time server 설정 전에 우선 알아야 할 것이 _**Stratum**_ 이라는 의미입니다. _**Stratum**_은 지층이라는 의미로, NTP protocol은 피라미드 형식의 구성으로 이루어져 있기 때문에, _**Stratum**_ 0은 피라미드의 꼭대기라고 비유할 수 있습니다.

_**Stratum**_ 0은 _**primary reference clock**_ 이라고 부르며, NTP protocol과는 상관이 없습니다. 즉 직접적으로 시간 서비스를 하는 것은 아니며, _**Stratum**_ 1로 시간을 전송하는 장비들을 말하며 _**primary reference clock**_ 장비에는 _GPS_, _세슘 원자 시계_ 등이 있습니다.

보통 _**Stratum**_ 1 level의 서버들은 _**primary reference clock**_에서 시간을 동기화 하여 서비스를 하며, NTP에서 최상위층이라고 생각하면 됩니다.

다만, _**Stratum**_ 1 level의 서버들은 client들이 _**Stratum**_ 1 서버에서 동기화를 하면 시간이 더욱 정확할 것이라는 생각으로 _**Stratum**_ 1 서버들을 설정하여 서비스 부하가 높아져서 _**Stratum**_ 2 서버들에만 접근을 허가하고 access를 막아 놓은 경우가 대부분 입니다.

또한, NTP 구성 목적이 대부분 정확한 시간 보다는 시간의 동기화에 있기 때문에 꼭 최상위 _**stratum**_에 동기화를 할 이유가 별로 없기 때문에 _**Stratum**_ 2 정도에 sync를 하는 것을 권장 합니다.

NTP protocol과 서비스에 대한 자세한 설명은 [http://time.ewha.or.kr/](http://time.ewha.or.kr/) 을 참조 하십시오.

### 2. Chrony를 이용한 시간 동기화

_**chrony**_는 NTP와 동일하게 Time service를 하지 않고 서버의 시간 동기화를 하기만 하려고 해도 기본적으로 daemon으로 구동을 해야 합니다.

안녕 리눅스 3은 설치시에 기본적으로 _Stratum 2_ 또는 _Stratum 3_으로 구성된 _CentOS NTP pool_에 등록되어 있는 외부 Time server로 부터 시간을 동기화 하도록 되어 있습니다. 즉, 단순히 서버의 시간 동기화를 위해서라면, 별도의 설정을 건드릴 필요 없이 _**chronyd**_만 실행을 시켜 주면 된다는 의미입니다.

만약, _**chrony**_가 설치 되어 있지 않은 상태에서, _**chrony**_ 설치 후 시간 동기화를 하고 싶다면 다음 명령을 이용할 수 있습니다.

```bash
[root@an3 ~]$ yum remove ntp          # ntp package가 설치 되어 있다면 삭제해야 함
[root@an3 ~]$ yum install chrony
[root@an3 ~]$ service chronyd enable  # booting 시에 구동. 또는 'ntsysv-systemd' 명령을 실행해서 chronyd 체크
[root@an3 ~]$ service chronyd restart # chronyd 시작
```

또한, 기본으로 설정이 되어 있는 _CentOS NTP pool_의 time server가 아니라 다른 Time server에서 동기화를 하고 싶다면 _**/etc/chrony/chrony.conf**_ 에서 _**server**_ 지시자에 원하는 Time server를 등록하면 됩니다.

```text
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```

만약 daemon으로 구동을 하기 싫다면, _**chronyd**_ process를 중지시키고, _**ntpdate**_를 이용하여 manual로 동기화 할 수 있습니다.

```bash
[root@an3 ~]$ service chronyd stop
[root@an3 ~]$ service chronyd disable
[root@an3 ~]$ yum install ntpdate
[root@ane ~]$ service ntpdate enable
[root@an3 ~]$ # 또는
[root@an3 ~]$ ntsysv-systemd
```

위와 같이 명령을 실행을 하면, chronyd는 구동하지 않으며, booting 시에 _**ntpdate**_로 시간 동기화를 하게 됩니다. 주의할 점은 부팅시에 ntpdate는 /etc/chrony/chrony.conf 또는 /etc/ntp/ntp.conf의 _**server**_로 등록되어진 time server로 부터 동기화를 하기 때문에 chonry.conf 또는 ntp.conf에 time server가 설정 되어 있어야 합니다. \(chrony.conf가 ntp.conf보다 우선 합니다.\)

또한, 원할 때 `ntpdate` 명령을 실행함으로서 동기화를 할 수 있습니다.

```bash
[root@an3 ~]$ ntpdate 0.centos.pool.ntp.org
```

참고로, ntpd와는 달리 chronyd가 구동중이라도 _**ntpdate**_ 실행이 가능 합니다.

### 3. Time server 구성

많은 수의 서버를 관리할 경우, 모든 서버를 외부의 Time server를 이용하거나 또는 _CentOS NTP Pool_의 Time server를 이용할 경우, 서버마다 시간 차가 발생할 수 있습니다.

이럴 경우에는 local network에 Time server를 구성을 하고, 서버들이 이 Time server를 바라보게 하여 운영을 하는 것이 훨씬 더 좋습니다.

구성도를 보자면 다음과 같은 구성을 하는 것이 일반적 입니다.

![&#xB85C;&#xCEEC; Time server &#xAD6C;&#xC131;](../.gitbook/assets/stratum%20%281%29.jpg)

내부에 fail over를 위해 Stratum 3 level의 Time server 2대를 구성하고, private network 안에 있는 server/client 들이 이 time server를 통해서 시간 동기화를 하도록 구성을 하도록 합니다.

그리고, 2대의 Time server 간에는 peer 구성을 하여 서로 동기화를 하게 할 수 있지만, 제 개인적인 견해로는 Time service 특성상 peer 구성 보다는 그냥 master 2대로 구성하는 것이 관리상 더 편했던 것 같습니다. 그래서 여기서는 peer 구성은 하지 않고 그냥 time server 2대를 독립적으로 구성하되, sync할 stratum 2 level의 서버를 동일하게 지정하여 peer 설정을 한 것과 비슷하게 구성을 할 것입니다.

여기서는 다음의 환경으로 Time Server 1과 Time Server 2를 구성하는 예로 설명을 합니다.

* local network : 10.0.0.0/8
* Time Server 1 IP: 10.10.0.1
* Time Server 2 IP: 10.10.10.1
* Time Server 1과 2는 Stratum 3로 구성

Time server를 운영하는 데 있어 사용되는 resource는 극히 적습니다. 실제로 Stratum 2 서버들 중에는 현재 486 machine으로 동작하는 경우도 있습니다. 그러므로, DNS나 DHCP 서버에 Time server 설정을 하는 것을 권장 합니다.

#### 3.1 /etc/chrony/chrony.conf

상단의 구성 처럼 2대의 Time server를 구성할 경우, 두대의 Time server간의 동기화를 위해서 peer 설정을 하게 됩니다. 하지만, Time server의 source stratum이 동일할 경우 굳이 peer 설정이 의미가 별로 없기 때문에 그냥 독립적인 time server 2대를 구성하는 것도 문제가 없습니다.

일단 안녕 리눅스 3의 chrony.conf에 등록이 되어 있는 Time server들은 대부분 Stratum 2 level 입니다. 그러므로 굳이 이를 변경할 필요는 없습니다. 굳이 원하는 time server가 있다면 찾아서 변경해 주시면 됩니다.

[http://time.ewha.or.kr/domestic.html](http://time.ewha.or.kr/domestic.html) 를 참고 하여 Stratum 2 3개 정도, Stratum 3 1개 정도를 선택 합니다.

_**chrony**_는 _**NTP**_와는 달리 기본 설정에서 _access_ 설정을 하지 않으면 UDP 123번 port를 listen 하지 않습니다. 즉, client mode로만 동작을 한다는 의미입니다. 그러므로 _**chrony.conf**_에 _**allow**_ 설정을 해 주어야 Time service가 가능합니다.

그리고 _**local**_ 지시자로 이 서버가 Stratum 3임을 announce 하도록 설정 합니다.

```text
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

위와 같이 설정이 되어 있으면 됩니다. \(기본 설정 파일에서 _**allow**_와 _**local**_만 추가가 됩니다.\)

#### 3.2 방화벽 설정

방화벽이나 subnet 구간에 switch ACL이 있다면 time server 1\(10.10.0.1\)과 time server 2\(10.10.10.1\)의 UDP 123번 포트를 open 해 주어야 합니다.

다음은 time server 자체에서 방화벽\(oops-firewall 또는 firewalld\)이 운영중일 경우 입니다. private network이라서 방화벽을 구동하고 있지 않다면 무시 하십시오.

time server에 _**oops-firewall**_이 실행 되고 있다면 _**/etc/oops-firewall/filter.conf**_의 _**UDP\_ALLOWPORT**_에 _123_을 추가해 주십시오.

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

time server에 _**firewalld**_가 실행이 되고 있다면 다음과 같이 하십시오.

```bash
[root@an3 ~]$ firewall-cmd --permanent --zone=public --add-port=123/udp
```

#### 3.3 daemon 구동 및 서비스 확인

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

#### 3.4 client 설정

Time server가 _**NTP**_로 구축이 되었더라도, client에서 _**chrony**_를 사용해도 무방 합니다.

**3.4.1 /etc/chrony/chrony.conf**

client 설정에서는 _**server**_ 지시자만 새로 만든 time server 1과 time server 2를 지정해 주면 되며, 그 외에는 수정할 필요가 없습니다.

```text
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 10.10.0.1 iburst
server 10.10.10.1 iburst
```

**3.4.2 방화벽 설정**

client에 _**oops-firewall**_이 실행되고 있다면 _**OUT\_UDP\_HOSTPERPORT**_에 time server의 123번 포트를 설정해 주십시오. private network이라서 _**oops-firewall**_을 구동하고 있지 않다면 무시 하십시오.

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
OUT_UDP_HOSTPERPORT = 10.10.0.1:123 10.10.10.1:123

[root@an3 ~]$ oops-firewall -v  # oops-firewall 재구동
[root@an3 ~]$
```

**3.4.3 chrony 재시작 및 확인**

_**chrony**_를 재시작 한 후, _**chronyc**_를 이용하여 확인을 합니다.

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
^* 10.0.0.1                      3   6    77    14  +1656us[+1597us] +/-   1ms
^+ 10.10.10.1                    3   6    77    12  -2475us[-2475us] +/-   1ms
```

### 4. 윤초 대응

> _**윤초\(閏秒\)**_란 협정 세계시에서 기준으로 삼고 있는 세슘 원자시계와 실제 지구의 자전·공전 속도를 기준으로 한 태양시의 차이로 인해 발생한 오차를 보정하기 위하여 추가하는 1초이다. 12월 31일의 마지막에 추가하거나, 혹은 6월 30일의 마지막에 추가한다.\(출처: 위치백과\)

보통 _**윤초**_가 추가되게 되면, RHEL/CentOS에서는 _**tzdata**_ package를 업데이트 하여 시스템에 반영을 하게 됩니다. \(/etc/localtime을 윤초가 반영된 파일로 갱신 시켜 glibc가 reload 하도록 합니다.\)

그런데 문제는 시간 동기화 데몬\(chrony, npt, ptp 등\)을 운영하는 경우 특정 버그로 인하여 문제가 되는 경우가 발생할 수있습니다. [대표적인 사건으로 2016.6.30일의 윤초 추가시에 ~~time server, 리눅스 커널 버그와 JVM 버그의 3단 컴비네이션~~으로 인하여 CPU 100% 사용률을 가지게 되는 장애를 발생시키는 사건이 있었습니다. \(linkedIn, Reddit 등..\)](https://translate.google.com/translate?hl=ko&sl=ja&tl=ko&u=https%3A%2F%2Fsrad.jp%2F~marusa%2Fjournal%2F593599%2F)

그래서 이를 원천적으로 회피하기 위한 기술로 Google에서 [Leap Smear](https://googleblog.blogspot.kr/2011/09/time-technology-and-leaping-seconds.html) 기법을 발표하고, _**Chrony**_에서 이를 설정 하는 방법을 기술 합니다.

이 설정을 사용하기 위해서는 _**chrony**_ 버전이 2.0 보다 높아야 합니다. CentOS/RHEL 7이 처음 출시 되었을 때는 _**chrony**_ 1.3을 탑재하고 있었으므로, yum을 이용하여 최신 버전으로 업데이트를 해 주어야 합니다.

또한, 따로 time server를 운영하고 있지 않다면 모든 chrony client 설정에 추가를 해 주어야 합니다. 즉, 서버가 많다면 straum 3정도의 time 서버를 운영하시는 것을 권장 합니다.

_**/etc/chrony/chrony.conf**_ 에 다음의 설정을 추가 하십시오. \(RHEL 또는 CentOS에서는 _**/etc/chrony.conf**_ 입니다.\)

```php
# 윤초의 모드 설정. 클러킹 프로세스를 비활성화
leapsecmode slew
# slew 모드에서의 동기화 속도. slew 속도 1000ppm. 1 초 차이를
# 17 시간 34 분 동안 분할 동기화.
maxslewrate 1000
# 장기간 오프라인되어 후에도 순조롭게 시간 동기화.
smoothtime 400 0.001 leaponly
```

_**leapsecmode**_ 지시자는 평활화 프로세스\(smoothing process\)를 재설정하는 클럭 처리를 비활성화 합니다. _**maxslewrate**_ 지시자는 local 시계의 선회\(slewing\) 속도를 1000ppm으로 제한하여 로컬의 정정이 시작되고 끝날 때 평활화 프로세스\(smoothing process\)의 안전성을 향상시킵니다. _**smoothtime**_ 지시자는 서버 시간에 대한 평활화 프로세스\(smoothing process\)를 활성화 합니다.

평활화 서비스\(Smoothing process\)에 대해서는 [Leap Smear](https://googleblog.blogspot.kr/2011/09/time-technology-and-leaping-seconds.html) 문서를 참고 하시고, chorny 설정에 대해서는 [chrony.conf.5 man page](https://chrony.tuxfamily.org/doc/2.4/chrony.conf.html)를 참고 하십시오.

