# Domain 위임

도메인 위임이라는 의미는 서브 도메인의 권한을 다른 네임서버에게 준다는 의미 입니다.

예를 들어, _**a.com**_ 도메인의 권한을 가지고 있는 name server에서 _**son.a.com**_의 권한을 B 라는 네임 서버에게 위임을 했다면, B 라는 네임 서버는 _**son.a.com**_의 서브 도메인을 관리할 수 있다는 의미 입니다.

도메인 위임에는 sub domain의 위임과 inverse domain의 위임을 주로 하게 되며, 여기서는 이 둘을 모두 다루도록 합니다.

## 5.7.1 Sub domain 위임

여기서는 _**a.com**_ 도메인을 가지고 있는 네임 서버에서, _**son.a.com**_을 위임하는 예를 듭니다. 도메인 위임은 모두 _**a.com**_의 zone file 내에서 하게 되며, _**위임**_은 _**NS**_ record를 이용하여 설정을 합니다.

```text
@ORIGIN "a.com"
son             IN   NS  ns.son
                IN   NS  ns2.son
;
; ns.son glue record information
ns.son          IN   A   10.0.0.1
ns2.son         IN   A   10.0.0.2
```

위의 설정은, 다음의 의미를 가집니다.

1. son.a.com 은 ns.son.a.com과 ns2.son.a.com에서 관리를 한다.
2. ns.son.a.com 의 ip는 10.0.0.1 이고, ns1.son.a.com의 ip는 10.0.0.2 이다.
3. 이 외의 정보는, ns.son.a.com 또는 ns2.son.a.com 이 관리를 한다.

즉, 위임을 하는 서버에서는 son.a.com을 위임할 서버의 정보만 가지게 되고, 나머지 son.a.com에 대한 관리는 위임한 서버\(ns.son, ns2.son\)에서 하게 됩니다. 즉, ns.son.a.com과 ns2.son.a.com에서는 _**son.a.com**_을 ORIGIN으로 하는 도메인에 대한 announcing이 가능 하다는 의미가 됩니다.

위임 시에 주의 할 것은, _**NS**_ record 또는 _**MX**_ record에 지정되는 도메인은 _**glue domain**_이라고 하여, 절대적으로 _**A record**_만을 사용할 수 있습니다. 즉, 위에서 NS record로 지정된 _**ns.son**_의 경우,

```text
ns.son          IN   CNAME  ns.b.com.
```

위와 같이 CNAME을 사용하면 절대 안되며, 문제를 야기할 수 있습니다.

## 5.7.2 inverse domain 위임

일단, 기본적으로 inverse domain의 최소 단위는 C class 입니다. 왜나하면, _**in-addr.arpa**_의 표현의 경우 domain의 자리수를 이용하기 때문 입니다. 예를 들어, C class의 경우에는 제일 마지막 자리를 제거한 채 사용을 하고, B class의 경우 뒤에서 두 자리를 제거한 채 사용을 하기 때문 입니다.

```text
* 192.168.0.0/24 의 ORIGIN : 0.168.192.in-addr.arpa
* 192.168.0.0/16 의 ORIGIN : 168.192.in-addr.arpa
```

그러므로, 예를 들어 192.168.0.0 ~ 192.168.255.255 의 권한을 가진 서버에서 192.168.10.0/23 \(192.168.10.0 ~ 192.168.11.255\) 을 위임을 하려면 C class로 두번을 해야 합니다.

```text
10            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
11            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
```

그렇다면, 192.168.0.1 을 위임 하려면 어떻게 될까요? zone file에서는 LHS\(left hand side\)의 값이 dot으로 끝나지 않을 경우에는 값 뒤에 origin이 붙게 되어 있습니다. 그러므로, origin을 제외한 값이 LHS의 값으로 지정이 되면 되는 것이죠.

그럼, 192.168.0.1의 inverse domain은 1.0.168.192.in-addr.arpa 입니다. 그리고 orign은 b class 이기 때문에 168.192.in-addr.arpa가 되므로, 아래와 같이 설정을 해야 합니다.

```text
1.0           IN    NS  ns.b.com.
```

이해가 되시는가요?

그럼, 192.168.0.0/24 의 권한을 가지고 있을 경우, 192.168.0.8/29\(192.168.0.8 ~ 192.168.0.15\)의 권한을 위임하고자 할 때는 B class에서 C class 단위로 위임을 하듯이 8~15 까지 8개의 주소에 대하여 각각 위임을 해 줘야 합니다.

```text
8             IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
9             IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
10            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
11            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
12            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
13            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
14            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
15            IN    NS   ns.b.com.
              IN    NS   ns2.b.com.
```

이렇게 부분 위임의 경우에는 설정이 엄청나게 복잡해 지게 됩니다. 그래서 bind의 경우에는 $GENERATE 지시자라는 것을 지원합니다. 위의 설정은 아래와 같이 요약이 가능 합니다.

```text
$GENERATE 8-15 $    IN  NS  ns.b.com.
                    IN  NS  ns2.b.com.
```

여기서 한 경우를 더 해 보자면, 8~15 까지를 위임을 받은 ns.b.com 에서는 다음과 같이 설정을 하게 됩니다.

_**named.conf:**_

```text
zone "0.168.192.in-addr.arpa" IN {
    type master;
    file "reverse/192.168.0.8-15.zone";
    allow-update { none; };
}
```

_**192.168.0.8-15.zone:**_

```text
@               IN  SOA     ns.b.com.    hostmaster.b.com. (
                        2016101600  ; serial
                        10800       ; refresh
                        3600        ; retry
                        604800      ; expire
                        86400        ; ttl
                        )
;
; Name Server
;
                IN  NS      ns.b.com.
                IN  NS      ns2.b.com.
;
8               IN  NS      192-168-0-8.b.com.
9               IN  NS      192-168-0-8.b.com.
10              IN  NS      192-168-0-10.b.com.
11              IN  NS      192-168-0-11.b.com.
12              IN  NS      192-168-0-12.b.com.
13              IN  NS      192-168-0-13.b.com.
14              IN  NS      192-168-0-14.b.com.
15              IN  NS      192-168-0-15.b.com.
```

이렇게 설정이 되었을 경우 문제가 하나 발생하게 됩니다. 예를 들어 ns.b.com 으로 192.168.0.1 에 대한 질의를 하게 되면, ns.b.com에서 0.168.192.in-addr.arpa 의 설정을 해 두었기 때문에, 설정이 되지 않은 inverse domain이라는 응답이 뜨기 됩니다. 그러므로, ns.b.com에서는 권한이 없는 192.168.0.8/29\(192.168.0.8 ~ 192.168.0.15\)를 제외한 영역에 대하여 위임을 한 서버에게 아래와 같이 다시 위임을 해 주어야 합니다.

_**192.168.0.8-15.zone:**_

```text
@               IN  SOA     ns.b.com.    hostmaster.b.com. (
                        2016101600  ; serial
                        10800       ; refresh
                        3600        ; retry
                        604800      ; expire
                        86400        ; ttl
                        )
;
; Name Server
;
                IN  NS      ns.b.com.
                IN  NS      ns2.b.com.
;
$GENERATE 0-7  $    IN  NS  ns.a.com.
                    IN  NS  ns2.a.com.
;
8               IN  NS      192-168-0-8.b.com.
9               IN  NS      192-168-0-8.b.com.
10              IN  NS      192-168-0-10.b.com.
11              IN  NS      192-168-0-11.b.com.
12              IN  NS      192-168-0-12.b.com.
13              IN  NS      192-168-0-13.b.com.
14              IN  NS      192-168-0-14.b.com.
15              IN  NS      192-168-0-15.b.com.
;
$GENERATE 16-255 $  IN  NS  ns.a.com.
                    IN  NS  ns2.a.com.
```

