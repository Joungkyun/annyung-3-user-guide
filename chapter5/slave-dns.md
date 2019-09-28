---
description: 이 챕터에서는 Chapter 5.2 신규 도메인 설정의 구성에 이어 Master/Slave 구조를 구성하는 것을 설명합니다.
---

# Slave DNS 구성

> **목차:**
>
> 5.3.1. Master / Slave 의 이해
>
> 5.3.2. Master 설정
>
> * 5.3.2.1 ACL 설정 
> * 5.3.2.2 Msater 옵션 설정
>
> 5.3.3 Slave 설정
>
> * 5.3.3.1 ACL 설정
> * 5.3.3.2 Slave 옵션 설정
> * 5.3.3.3 Slave zone 설정
> * 5.3.3.4 Slave 동기화 확인

## 5.3.1 Master / Slave 의 이해

DNS 등록을 하다 보면 _**1차 네임 서버**_, _**2차 네임 서버**_라는 용어가 나옵니다. 영어로는 _**Primary Namer server**_, _**Secondary Name server**_ 라고 표현을 합니다.

이런 표현 때문에 오해가 많이 생기는데, 대부분의 많은 사람들이 _**2차 네임 서버**_, _**Secondary Name server**_ 라는 표현을 backup server로 이해를 하고 있습니다. 즉, _**1차 네임 서버**_가 죽으면 _**2차 네임 서버**_가 동작할 것이다. 물론 틀린 말은 아닙니다만 맞는 말도 아닙니다.

이런 이름 때문에, DNS 구조에서 name server가 _**active-standby**_ 구조로 되어 있다고 오해하시는 분들이 꽤 많이 있습니다.

하지만, 실제 DNS 구조상에서 name server는 _**active-standby**_가 아니라 _**active-active**_ 구조로 동작을 하게 됩니다. 즉, _**2차 네임 서버**_든 _**Secondary Name server**_건 이름에 상관없이 _**NS**_ record에 등록이 되어 있으면 서비스 측면에서는 모두 _**active**_한 상태라는 의미입니다.

설정으로 예를 들어 봅니다.

```text
@       IN    NS   ns.domain.org.
        IN    NS   ns2.domain.org.
```

[_**Chapter 5.2.2.7 A\(Address\) & CNAME\(Canonical Name\) record**_](chapter5-2-add-domain.md) 에서 _**DNS RR**_ 이라는 것을 설명 했습니다. 설정된 대로 차례대로 응답을 한다는 의미 입니다.

_**A**_ record 뿐 아니라, DNS는 모든 record에 대해서 **RR\(Round Robin\)\***을 수행 합니다. 즉 _**NS**_ record라고 예외는 없습니다. 그러므로 위와 같이 설정이 되어 있다면 _**1차 네임 서버**_, _**2차 네임 서버**_라는 용어는 의미가 없다는 것을 알아 차릴 수 있을 겁니다. _**NS**_ record에 대한 요청이 올 때 마다 번갈아 가면서 나올테니 무엇이 _**1차**_이고 _**2차**_인지 구분이 안된다는 의미 입니다.

즉, 이런 DNS 특성을 제대로 구현 못한 application을 사용할 경우, name server중 1개가 죽는다면 최대 50% 정도의 서비스 실패가 발생할 수도 있다는 의미 입니다. \(물론 그렇게 간단하게 서비스가 되지 않는 것은 아닙니다. _**최대**_라는 점에 주목 하셔야 합니다. 그야말로 _**최악**_의 상태일때를 말하는 겁니다. 그러므로 DNS 관련 개발을 할 경우에는 retry를 해 주는 것이 매우 중요한 요소 입니다.\)

여기서 이렇게 장황하게 설명하는 이유는, DNS 구조상 _**Master**_와 _**Slave**_가 _**active-standby**_ 구조가 아님을 강조하기 위함입니다. _**NS**_ record에 지정되어 있는 name server는 서비스 상으로는 모두 _**active**_ 구조이고, 단지 _**Master**_ / _**Slave**_라는 용어는 name server 설정 관리의 위한 용어일 뿐임을 강조하기 위해서 입니다. 즉,

* _**Master**_ : 설정 변경의 주체
* _**Slave**_  : Master의 설정 변경 사항을 전달 받아 자동으로 반영

위의 구조가 _**Master**_/_**Slave**_를 설명하는 것입니다.

또한, 중요한 것은, _**Master/Slave**_ 구조에서 Data가 동기화 되는 것은 zone database 만 가능 합니다. 즉, _**named.conf**_에서 변경되는 내역은 _**slave**_에 반영이 되지 않습니다.

## 5.3.2 Master 설정

_**bind**_가 _**slave**_ 서버로 zone file의 변경 사항을 통지 할 수 있도록 설정을 해 주어야 하고, zone file 변경 내역을 전송할 수 있도록 해 주어야 합니다. 변경 사항을 통지 하는 것을 _**notify**_라고 하며, 변경 내용 전송을 _**zone transfer**_라는 용어를 사용하게 되니, 숙지 하십시오.

### 5.3.2.1 ACL 설정

먼저, _**zone transfer**_를 위해서 _**/var/named/etc/named.acl.conf**_에 ACL group을 생성 합니다.

```text
acl DNSLIST {
    111.112.113.10;
    111.112.113.11;
};
```

_**NS**_ record로 지정되어 있는 모든 name server의 IP를 등록 하십시오.

### 5.3.2.2 Msater 옵션 설정

_**/var/named/etc/named.conf**_의 _**optoions**_에서 다음의 설정을 추가/변경 하십시오.

```text
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

안녕 리눅스의 _**bind**_의 경우, 기본으로 _**notify**_는 off 되어 있고, _**allow-tarnsfer**_는 localhost만 되도록 되어 있으므로, 위와 같이 변경해 주어야 합니다.

위의 설정이 완료 되었으면, 기본적으로 _**Master**_의 준비는 완료된 상태 입니다. 이제 변경된 설정값을 실행중인 daemon에 반영하기 위하여 _**bind**_를 reload 해 줍니다.

```bash
[root@ns etc]$ rndc reload
or
[root@ns etc]$ service named reload
```

_**Master**_ 설정에서 마지막으로 기억할 것은, zone file 수정 후에 _**SOA**_ 영역의 _**Serial**_을 변경해 주어야 _**zone transfer**_가 된다는 것을 꼭 명심해야 합니다.

## 5.3.3 Slave 설정

### 5.3.3.1 ACL 설정

_**Master**_와 동일하게 _**/var/named/etc/named.acl.conf**_에 _**DNSLIST**_ acl group을 생성 합니다.

```text
acl DNSLIST {
    111.112.113.10;
    111.112.113.11;
};
```

_**NS**_ record로 지정되어 있는 모든 name server의 IP를 등록 하십시오.

### 5.3.3.2 Slave 옵션 설정

_**Slave**_ 서버는 notify를 하지 않기 때문에, _**allow-transfer**_ 설정만 해 주도록 합니다.

_**/var/named/etc/named.conf**_의 _**optoions**_에서 다음의 설정을 추가/변경 하십시오.

```text
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

다음 역시 _**/var/named/etc/named.conf**_에 _**domain.org**_의 zone을 _**slave**_ type으로 추가 합니다.

```text
zone "domain.org" IN {
    type slave;
    file "slave/domain.org-slave.zone";
    master { 111.112.113.10; };
    allow-notify { 111.112.113.10; };
};
```

_**Slave**_ 정의는 _**Master**_와는 다른 준비가 필요 합니다.

일단, _**file**_에 정의되어 있는 zone file의 위치는 실행 되어 있는 _**bind**_가 write를 할 수 있는 권한이 있어야 합니다. 안녕 리눅스의 _**bind**_는 기본으로 _**named**_ 유저 권한으로 동작하므로, _**named**_ 유저에게 쓰기 권한이 있어야 합니다. 그러므로 안녕 리눅스의 _**bind**_ package에서 미리 권한을 맞추어 놓은 _**/var/named/zone/slave**_ 디렉토리를 이용하도록 하는 것입니다. 만약 다른 배포본이나, 직접 bind를 설치해서 사용하는 경우에는 _**file**_ 옵션에 지정된 파일을 _**bind**_ process가 control 할 수 있는지 여부를 확인 해야 합니다.

_**master**_ 옵션은 이 _**Slave**_ 영역의 _**Master**_ 서버의 정보를 지정 합니다. _**Master**_ 서버의 IP를 지정하며 됩니다.

_**allow-notify**_ 옵션 역시 _**Master**_ 서버의 IP를 지정 합니다.

### 5.3.3.4 Slave 동기화 확인

zone 설정을 마쳤다면, _**Master**_에서 했던 것 처럼 _**bind**_ process를 reload 해 줍니다.

```bash
[root@ns2 etc]$ rndc reload
or
[root@ns2 etc]$ service named reload
```

다음, _**/var/named/zone/slave/domain.org-slave.zone**_ 파일이 생성이 되었는지 확인을 합니다. 만약 생성이 되지 않았다면 _**/var/named/zone/slave/**_ 디렉토리의 권한 문제일 경우가 많습니다. 권한에 문제가 없을 경우에는 _**/var/log/named/named.log**_를 확인해서 어떠한 오류가 발생하고 있는지 확인해 보아야 합니다.

만약 transfer가 잘 되었다면 log에 다음과 같은 라인이 기록이 됩니다.

```text
12-Feb-2017 19:47:54.234 zone domain.org/IN: Transfer started.
12-Feb-2017 19:47:54.242 transfer of 'domain.org/IN' from 111.112.113.10#53: connected using 111.112.113.11#53768
12-Feb-2017 19:47:54.267 zone domain.org/IN: transferred serial 2017011500
12-Feb-2017 19:47:54.268 transfer of 'domain.org/IN' from 111.112.113.10#53: Transfer completed: 1 messages, 11 records, 123 bytes, 0.025 secs (51320 bytes/sec)
```

