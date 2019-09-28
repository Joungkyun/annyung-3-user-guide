# Inverse Domain 설정

_**Inverse domain**_은 IP 주소에 할당된 domain을 의미 합니다. 좀더 쉽게 말하면, IP에 해당하는 domain을 역으로 찾을 수 있도록 합니다. 즉, 일반적인 name service는 domain에 IP를 할당하는 구조이지만, _**inverse domain**_은 IP 주소에 domain을 할당하는 것 입니다.

_**inverse domain**_은 다른 표현으로 _**reverse mapping**_ 이라고도 하고 _**reverse DNS lookup**_이라는 표현도 사용을 합니다.

_**inverse domain**_ 검색은 다음과 같이 할 수 있습니다.

```bash
[root@an3 ~]$ nslookup
> 172.16.10.15
Server:         172.16.1.1
Address:        172.16.1.1#53

15.10.16.172.in-addr.arpa       name = kldp.org.
[root@an3 ~]$
[root@an3 ~]$
[root@an3 ~]$ dig -x 172.16.10.15

; <<>> DiG 9.9.4-geoip-1.4-RedHat-9.9.4-38.an3.2 <<>> -x 172.16.10.15
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 761
;; flags: qr aa rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;15.10.16.172.in-addr.arpa.     IN      PTR

;; ANSWER SECTION:
15.10.16.172.in-addr.arpa. 0    IN      PTR     kldp.org.

;; Query time: 1 msec
;; SERVER: 172.16.1.1#53(172.16.1.1)
;; WHEN: 월  4월 17 22:26:08 KST 2017
;; MSG SIZE  rcvd: 65
```

_**Inverse domain**_에 대한 자세한 설명은 [http://krnic.net/jsp/resources/inverseInfo.jsp](http://krnic.net/jsp/resources/inverseInfo.jsp) 문서를 참조 하십시오.

_**Inverse domain**_은 대부분 관리의 목적으로 많이 사용이 되므로, 실제로 대부분의 IP 주소들이 _**inverse domain**_ 설정이 되어 있지 않습니다. 하지만, SMTP 서버를 운영할 경우에, SMTP 서버의 경우에는 _**inverse domain**_ 설정이 되어 있지 않으면 _**SPAM**_ 발송 서버로 등록이 되기 쉽기 때문에 _**SMTP**_ 서버에는 거의 필수로 설정을 해 주야 합니다.

## 5.4.1 Inverse domain 권한 신청

inverse domain 역시 domain 처럼 상위 DNS에서 위임을 받아야 public 하게 서비스가 가능 합니다. 하지만 domain 처럼 비용을 지불하지는 않습니다.

일반적으로 inverse domain은 ip에 어떠한 행동을 하는 것이기 때문에, IP 할당 기관으로 부터 해당 IP 대역에 대한 권한을 위임 받게 됩니다. 국내의 IP 할당 기관은 _**KISA**_에서 담당하고 있으며, 대부분의 경우는 ISP의 회선을 빌려 쓰기 때문에, 회선 대여 ISP에 할당 받은 IP에 대한 inverse domain 권한 신청을 하면 됩니다.

> 독자적인 회선과 상면을 구축한 인터넷 사업자 중에는 ISP를 통해서 IP를 할당 받지 않고 _**KISA**_에서 직접 할당 받는 경우가 있습니다. 이러한 사업자들을 _**독립사업자**_라고 하며, _**NAVER**_, _**NEOWIZ**_, _**KAKAO**_ 등의 인터넷 사업자가 있으며, _**KT**_, _**LG U+**_, _**SK Broadband**_ 등과 같이 할당 받은 IP주소를 다른 업체에 재할당 해 주는 ISP들은 _**관리대행자**_ 라고 합니다.

여기서는, _**KT**_ 에서 14.63.245.0/24 네트워크를 할당 받았다고 예를 들어 설명을 합니다.

먼저, _**KT**_의 기술 지원 부서로 할당 받은 14.63.245.0/24 네트워크에 대한 inverse domain 설정을 합니다. IDC 계약자의 경우에는 IDC 고객 사이트의 계정으로 로그인을 하면 웹상에서 요청을 할 수 있도록 되어 있습니다. 이런 시스템이 없을 경우에는 영업 사원을 통하여 신청을 할 수도 있습니다.

_**inverse domain**_의 형식은 다음과 같으며 최소 단위는 _**C class**_ 입니다. 14.63.245.0/24 에 대한 _**inverse domain**_은 다음과 같이 표현 합니다.

```text
245.64.14.in-addr.arpa
```

C class 이기 때문에, 마지막 자리를 제외하고, 나머지 세자리 순서를 뒤집어서 기록한 다음, _**.in-addr.arpa**_를 postfix로 붙여 줍니다. B class 라면 뒤의 두자리를 제외하면 됩니다.

```text
64.14.in-addr.arpa
```

일단, inverse domain 권한 위임 요청은 아래와 같이 하면 됩니다.

> 14.64.245.0/24 네트워크에 대한 inverse domain 권한을 다음의 DNS로 위임해 주세요. Primary DNS : 14.64.245.10 Secondary DNS: 14.64.245.11

또는

> 245.64.14.in-addr.arpa에 대한 inverse domain 권한을 다음의 DNS로 위임해 주세요. Primary DNS : 14.64.245.10 Secondary DNS: 14.64.245.11

inverse domain 권한을 위임 받기 위해서는 직접 관리할 수 있는 DNS 서버가 필요 합니다. 권한이 없는 DNS에 의미없이 할당을 했을 경우에는 문제가 발생할 수도 있습니다.

위임이 되었는지 여부는 public DNS를 이용하여 다음과 같이 확인이 가능 합니다.

```bash
[root@an3 ~]$ nslookup --query=ns 245.64.14.in-addr.arpa
Server:     kns.kornet.net
Address:    168.126.63.1#53

245.64.14.in-addr.arpa       nameserver = ns.mydomain.org.
245.64.14.in-addr.arpa       nameserver = ns2.mydomain.org.
```

위와 같이 query를 해서, 내가 신청한 도메인이 나온다면 해당 네트워크에 대한 inverse domain 권한을 위임 받은 것입니다.

## 5.4.2 Inverse domain 설정

### 5.4.2.1 Zone 설정

_**inverse domain**_도 일반 domain과 동일하게 먼저 zone 설정을 합니다.

_**/var/named/etc/named.user.conf**_ 에 추가할 inverse domain zone을 정의 합니다.

```text
zone "256.64.14.in-addr.arpa" IN {
    type master;
    file "14.64.256.0.zone";
    allow-update { none; };
};
```

Slave 구성을 하고 싶다면, [5.3 Slave DNS 구성](https://joungkyun.gitbooks.io/annyung-3-user-guide/content/slave-dns.html) 문서를 참고하여 동일하게 구성해 주면 됩니다.

### 5.4.2.2 Zone file 설정

기본적인 zone file 형식은 동일합니다. 다만, 역방향 탐색에 필요한 record는 _**A**_ record 대신 _**PTR**_ record를 사용 합니다.

```text
; default ttl is 1 day.
$TTL 86400
@               IN  SOA ns.mydomain.org. admin.mydomain.org. (
                2017011500 ; serial
                10800      ; refresh
                3600       ; retry
                604800     ; expire
                86400      ; negative ttl
                )

                IN  NS      ns.mydomain.org.
                IN  NS      ns2.mydomain.org.
;
; Defines in.addr-arpa
1               IN  PTR     gateway.mydomain.com.
10              IN  PTR     ns.mydoamin.com.
11              IN  PTR     ns2.mydomain.com.
20              IN  PTR     www.mydomain.com.
```

PTR record 의 경우에는 각 IP 주소의 마지막 자리에 대해서 domain 이름을 매핑 하고 있습니다. 예를 들어

```text
1               IN  PTR     gateway.mydomain.com.
```

의 설정은 14.64.256.1 IP에 gateway.mydomain.com 을 mapping 하라는 의미 입니다.

