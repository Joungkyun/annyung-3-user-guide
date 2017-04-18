## Chapter 5.7 Domain 위임

>목차
5.7.1 Sub domain 위임
5.7.2 inverse domain 위임

<br>

도메인 위임이라는 의미는 서브 도메인의 권한을 다른 네임서버에게 준다는 의미 입니다.

예를 들어, ***a.com*** 도메인의 권한을 가지고 있는 name server에서 ***son.a.com***의 권한을 B 라는 네임 서버에게 위임을 했다면, B 라는 네임 서버는 ***son.a.com***의 서브 도메인을 관리할 수 있다는 의미 입니다.

도메인 위임에는 sub domain의 위임과 inverse domain의 위임을 주로 하게 되며, 여기서는 이 둘을 모두 다루도록 합니다.

### 5.7.1 Sub domain 위임

여기서는 ***a.com*** 도메인을 가지고 있는 네임 서버에서, ***son.a.com***을 위임하는 예를 듭니다. 도메인 위임은 모두 ***a.com***의 zone file 내에서 하게 되며, ***위임***은 ***NS*** record를 이용하여 설정을 합니다.

```
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

즉, 위임을 하는 서버에서는 son.a.com을 위임할 서버의 정보만 가지게 되고, 나머지 son.a.com에 대한 관리는 위임한 서버(ns.son, ns2.son)에서 하게 됩니다. 즉, ns.son.a.com과 ns2.son.a.com에서는 ***son.a.com***을 ORIGIN으로 하는 도메인에 대한 announcing이 가능 하다는 의미가 됩니다.

위임 시에 주의 할 것은, ***NS*** record 또는 ***MX*** record에 지정되는 도메인은 ***glue domain***이라고 하여, 절대적으로 ***A record***만을 사용할 수 있습니다. 즉, 위에서 NS record로 지정된 ***ns.son***의 경우,

```
ns.son          IN   CNAME  ns.b.com
```

위와 같이 CNAME을 사용하면 절대 안되며, 문제를 야기할 수 있습니다.
