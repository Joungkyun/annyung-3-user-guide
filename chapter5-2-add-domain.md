# Chapter 5.2 Bind 신규 도메인 설정

이 챕터는 ***annyung.oops.org*** 도메인을 bind에 추가하고 관리하는 것을 예로 듭니다.

## 5.2.1 domain zone 정의

***/var/named/etc/named.user.conf*** 에 추가할 domain zone을 정의 합니다.

```bind
zone "annyung.oops.org" IN {
    type master;
    file "annyung.oops.org.zone";
    allow-update { none; };
};
```

zone 정의에 대해서는 https://ftp.isc.org/isc/bind9/cur/9.9/doc/arm/Bv9ARM.ch06.html#zone_statement_grammar 문서에 자세하게 나와 있으니 참고 하시고, 위와 같이 최소한의 정의를 해 주면 domain을 lookup 하는데 문제가 없습니다.

위의 설정은, ***annyung.oops.org*** 도메인을 ***master***로 정의를 했으며, zone의 상세 설정은 ***/var/name/zone/annyung.oops.org.zone*** 파일에서 한다는 의미입니다.

## 5.2.2 zone 설정

zone을 정의하였으면, 그 다음 해당 zone에 대한 상세 설정을 합니다. 즉, zone을 정의 하였다는 것은 bind에서 운영할 도메인을 추가하였다는 의미이며, zone 설정은 추가한 도메인을 관리하기 위한 설정을 하는 것입니다. 예를 들어, 서브 도메인 추가 같은 설정을 의미합니다.

***/var/named/zone/annyung.oops.org.zone*** 파일을 생성하고 다음의 template으로 설정을 하도록 합니다.

```zone
$TTL 86400
@               IN  SOA ns.oops.org. admin.oops.org. (
                2017011500
                10800
                3600
                604800
                86400
                )

                IN  NS      ns.oops.org.
                IN  MX 10   mail
                IN  A       14.0.82.85

www             IN  CNAME   @
ftp             IN  CNAME   @
mail            IN  A       111.112.113.114
                IN  TXT     "v=spf1 include:an3.pkg.oops.org ~all"

``` 

