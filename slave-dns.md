# Chapter 5.3 Slave DNS 구성

>목차
5.3.1. Master / Slave 의 이해
5.3.2. Master 설정
 * 5.3.2.1 ACL 설정 
 * 5.3.2.2 Msater 옵션 설정
 
>5.3.3 Slave 설정
 * 5.3.3.1 ACL 설정
 * 5.3.3.2 Slave 옵션 설정
 * 5.3.3.3 Slave zone 설정
 * 5.3.3.4 Slave 동기화 확인


<br>


이 챕터에서는 ***[Chapter 5.2 신규 도메인 설정](chapter5-2-add-domain.md)***의 구성에 이어 Master/Slave 구조를 구성하는 것을 설명합니다.

## 5.3.1 Master / Slave 의 이해

DNS 등록을 하다 보면 ***1차 네임 서버***, ***2차 네임 서버***라는 용어가 나옵니다. 영어로는 ***Primary Namer server***, ***Secondary Name server*** 라고 표현을 합니다.

이런 표현 때문에 오해가 많이 생기는데, 대부분의 많은 사람들이 ***2차 네임 서버***, ***Secondary Name server*** 라는 표현을 backup server로 이해를 하고 있습니다. 즉, ***1차 네임 서버***가 죽으면 ***2차 네임 서버***가 동작할 것이다. 물론 틀린 말은 아닙니다만 맞는 말도 아닙니다.

이런 이름 때문에, DNS 구조에서 name server가 ***active-standby*** 구조로 되어 있다고 오해하시는 분들이 꽤 많이 있습니다.

하지만, 실제 DNS 구조상에서 name server는 ***active-standby***가 아니라 ***active-active*** 구조로 동작을 하게 됩니다. 즉, ***2차 네임 서버***든 ***Secondary Name server***건 이름에 상관없이 ***NS*** record에 등록이 되어 있으면 서비스 측면에서는 모두 ***active***한 상태라는 의미입니다.

설정으로 예를 들어 봅니다.

```
@       IN    NS   ns.domain.org.
        IN    NS   ns2.domain.org.
```

***[Chapter 5.2.2.7 A(Address) & CNAME(Canonical Name) record](chapter5-2-add-domain.md)*** 에서 ***DNS RR*** 이라는 것을 설명 했습니다. 설정된 대로 차례대로 응답을 한다는 의미 입니다.

***A*** record 뿐 아니라, DNS는 모든 record에 대해서 **RR(Round Robin)***을 수행 합니다. 즉 ***NS*** record라고 예외는 없습니다. 그러므로 위와 같이 설정이 되어 있다면 ***1차 네임 서버***, ***2차 네임 서버***라는 용어는 의미가 없다는 것을 알아 차릴 수 있을 겁니다. ***NS*** record에 대한 요청이 올 때 마다 번갈아 가면서 나올테니 무엇이 ***1차***이고 ***2차***인지 구분이 안된다는 의미 입니다.

즉, 이런 DNS 특성을 제대로 구현 못한 application을 사용할 경우, name server중 1개가 죽는다면 최대 50% 정도의 서비스 실패가 발생할 수도 있다는 의미 입니다. (물론 그렇게 간단하게 서비스가 되지 않는 것은 아닙니다. ***최대***라는 점에 주목 하셔야 합니다. 그야말로 ***최악***의 상태일때를 말하는 겁니다. 그러므로 DNS 관련 개발을 할 경우에는 retry를 해 주는 것이 매우 중요한 요소 입니다.)

여기서 이렇게 장황하게 설명하는 이유는, DNS 구조상 ***Master***와 ***Slave***가 ***active-standby*** 구조가 아님을 강조하기 위함입니다. ***NS*** record에 지정되어 있는 name server는 서비스 상으로는 모두 ***active*** 구조이고, 단지 ***Master*** / ***Slave***라는 용어는 name server 설정 관리의 위한 용어일 뿐임을 강조하기 위해서 입니다. 즉,

 * ***Master*** : 설정 변경의 주체
 * ***Slave***  : Master의 설정 변경 사항을 전달 받아 자동으로 반영
 
위의 구조가 ***Master***/***Slave***를 설명하는 것입니다.

또한, 중요한 것은, ***Master/Slave*** 구조에서 Data가 동기화 되는 것은 zone database 만 가능 합니다. 즉, ***named.conf***에서 변경되는 내역은 ***slave***에 반영이 되지 않습니다.

## 5.3.2 Master 설정

***bind***가 ***slave*** 서버로 zone file의 변경 사항을 통지 할 수 있도록 설정을 해 주어야 하고, zone file 변경 내역을 전송할 수 있도록 해 주어야 합니다. 변경 사항을 통지 하는 것을 ***notify***라고 하며, 변경 내용 전송을 ***zone transfer***라는 용어를 사용하게 되니, 숙지 하십시오.

### 5.3.2.1 ACL 설정 

먼저, ***zone transfer***를 위해서 ***/var/named/etc/named.acl.conf***에 ACL group을 생성 합니다.

```
acl DNSLIST {
    111.112.113.10;
    111.112.113.11;
};
```

***NS*** record로 지정되어 있는 모든 name server의 IP를 등록 하십시오.

### 5.3.2.2 Msater 옵션 설정

***/var/named/etc/named.conf***의 ***optoions***에서 다음의 설정을 추가/변경 하십시오.

```
options {
    ...
    allow-transfer {
        LocalAllow;
        DNSLIST;
    };
    ...
    notify yes;
    ...
};
```

안녕 리눅스의 ***bind***의 경우, 기본으로 ***notify***는 off 되어 있고, ***allow-tarnsfer***는 localhost만 되도록 되어 있으므로, 위와 같이 변경해 주어야 합니다.

위의 설정이 완료 되었으면, 기본적으로 ***Master***의 준비는 완료된 상태 입니다. 이제 변경된 설정값을 실행중인 daemon에 반영하기 위하여 ***bind***를 reload 해 줍니다.

```bash
[root@ns etc]$ rndc reload
or
[root@ns etc]$ service named reload
```

***Master*** 설정에서 마지막으로 기억할 것은, zone file 수정 후에 ***SOA*** 영역의 ***Serial***을 변경해 주어야 ***zone transfer***가 된다는 것을 꼭 명심해야 합니다.

## 5.3.3 Slave 설정

### 5.3.3.1 ACL 설정

***Master***와 동일하게 ***/var/named/etc/named.acl.conf***에 ***DNSLIST*** acl group을 생성 합니다.

```
acl DNSLIST {
    111.112.113.10;
    111.112.113.11;
};
```

***NS*** record로 지정되어 있는 모든 name server의 IP를 등록 하십시오.

### 5.3.3.2 Slave 옵션 설정

***Slave*** 서버는 notify를 하지 않기 때문에, ***allow-transfer*** 설정만 해 주도록 합니다.

***/var/named/etc/named.conf***의 ***optoions***에서 다음의 설정을 추가/변경 하십시오.

```
options {
    ...
    allow-transfer {
        LocalAllow;
        DNSLIST;
    };
    ...
};
```

### 5.3.3.3 Slave zone 설정

다음 역시 ***/var/named/etc/named.conf***에 ***domain.org***의 zone을 ***slave*** type으로 추가 합니다.

```
zone "domain.org" IN {
    type slave;
    file "slave/domain.org-slave.zone";
    master { 111.112.113.10; };
    allow-notify { 111.112.113.10; };
};
```

***Slave*** 정의는 ***Master***와는 다른 준비가 필요 합니다.

일단, ***file***에 정의되어 있는 zone file의 위치는 실행 되어 있는 ***bind***가 write를 할 수 있는 권한이 있어야 합니다. 안녕 리눅스의 ***bind***는 기본으로 ***named*** 유저 권한으로 동작하므로, ***named*** 유저에게 쓰기 권한이 있어야 합니다. 그러므로 안녕 리눅스의 ***bind*** package에서 미리 권한을 맞추어 놓은 ***/var/named/zone/slave*** 디렉토리를 이용하도록 하는 것입니다. 만약 다른 배포본이나, 직접 bind를 설치해서 사용하는 경우에는 ***file*** 옵션에 지정된 파일을 ***bind*** process가 control 할 수 있는지 여부를 확인 해야 합니다.

***master*** 옵션은 이 ***Slave*** 영역의 ***Master*** 서버의 정보를 지정 합니다. ***Master*** 서버의 IP를 지정하며 됩니다.

***allow-notify*** 옵션 역시 ***Master*** 서버의 IP를 지정 합니다.

### 5.3.3.4 Slave 동기화 확인

zone 설정을 마쳤다면, ***Master***에서 했던 것 처럼 ***bind*** process를 reload 해 줍니다.

```bash
[root@ns2 etc]$ rndc reload
or
[root@ns2 etc]$ service named reload
```

다음, ***/var/named/zone/slave/domain.org-slave.zone*** 파일이 생성이 되었는지 확인을 합니다. 만약 생성이 되지 않았다면 ***/var/named/zone/slave/*** 디렉토리의 권한 문제일 경우가 많습니다. 권한에 문제가 없을 경우에는 ***/var/log/named/named.log***를 확인해서 어떠한 오류가 발생하고 있는지 확인해 보아야 합니다.

만약 transfer가 잘 되었다면 log에 다음과 같은 라인이 기록이 됩니다.

```
12-Feb-2017 19:47:54.234 zone domain.org/IN: Transfer started.
12-Feb-2017 19:47:54.242 transfer of 'domain.org/IN' from 111.112.113.10#53: connected using 111.112.113.11#53768
12-Feb-2017 19:47:54.267 zone domain.org/IN: transferred serial 2017011500
12-Feb-2017 19:47:54.268 transfer of 'domain.org/IN' from 111.112.113.10#53: Transfer completed: 1 messages, 11 records, 123 bytes, 0.025 secs (51320 bytes/sec)
```

