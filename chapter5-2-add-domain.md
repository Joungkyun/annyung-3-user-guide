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

## 5.2.2 zone database 설정

zone을 정의하였으면, 그 다음 해당 zone에 대한 상세 설정을 합니다. 즉, zone을 정의 하였다는 것은 bind에서 운영할 도메인을 추가하였다는 의미이며, zone 설정은 추가한 도메인을 관리하기 위한 설정을 하는 것입니다. 예를 들어, 서브 도메인 추가 같은 설정을 의미합니다.

***/var/named/zone/annyung.oops.org.zone*** 파일을 생성하고 다음의 template으로 설정을 하도록 합니다.

참고로, zone file에서 주석은 ***세미콜론(;)***을 이용하여 처리 합니다.

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

### 5.2.2.1 도메인 origin

도메인 origin이라는 것은, zone block 시에 정의를 해 준 도메인을 zone 파일에서 domain origin 이라고 명칭 합니다. 위의 예에서는 ***annyung.oops.org***가 도메인 origin으로 사용이 되며, zone file 내부에서는 ***@*** 기호로 단축해서 사용을 할 수 있습니다. 예를 들어

```zone
www             IN  CNAME   @
```

위의 설정은, www.annyung.oops.org(***www***)를 annyung.oops.org(***@***)의 ***CNAME***으로 설정하라는 의미입니다.

또한, ***$ORIGIN*** 키워드를 이용하여 ***origin***을 변경할 수 있습니다.

```zone
www             IN  CNAME   @
$ORIGIN www.annyung.oops.org.
data            IN  CNAME   @  ; data.www.annyung.oops.org를 www.annyung.oops.org의 CNAME으로 설정
$ORIGIN annyung.oops.org.
data            IN  CNAME   @  ; data.annyung.oops.org를 annyung.oops.org의 CNAME으로 설정
```

### 5.2.2.2 도메인 표현 형식

도메인 표현 형식은 zone 파일 설정에서 아주 중요한 부분 입니다. 대부분의 DNS 설정 오류가 도메인 형식을 잘못 사용하여 발생을 하게 됩니다.

기본적으로, zone 파일에서 완전한 도메인을 표기할 때는 마지막에 ***annyung.oops.org.***와 같이 ***dot(.)***로 끝이 나야 합니다. 이름 마지막이 ***dot(.)***으로 끝나지 않을 경우에는 bind는 그 뒤에 ***origin***이 붙는 것으로 간주를 한다. 즉, ***www.annyung.oops.org***는 ***www.annyung.oops.org.annyung.oops.org***로 인식이 되는 것 입니다. 그러므로 다음의 설정에서

```zone
www             IN  CNAME   @
```

***www***는 ***dot(.)***으로 끝나지 않았기 때문에 내부적으로 ***www.annyung.oops.org***로 처리가 되는 것 입니다.

이 부분은 숙련되 엔지니어도 자주 하는 실수 영역이므로, zone databse 설정 시에는 이를 숙지하면서 작업을 해야 합니다.

### 5.2.2.3 zone database 설정 형식

zone file에서 작성하는 database의 형식은 다음과 같습니다.

```
[DOMAIN]           [TTL]  [CLASS] [RECORD]  [DOMIAN | IPADDRESS]
annyugn.oosp.org.  86400  IN      A         1.1.1.1
```

***TTL*** 필드는 생략이 가능하며, 이에 대해서는 ***TTL 설정*** 항목에서 다룰 것 입니다.

***CLASS***는 ***IN***, ***HS***, ***HESIOD***, ***CHAOS*** 등이 있지만, 대부분 ***IN*** 외에는 사용할 일이 거의 없습니다. 아주 특별한 설정이 아닌 이상 ***IN***만 사용한다고 생각하면 됩니다.

***RECORD***에는 자주 사용하는 것으로 ***SOA***, ***A***, ***AAAA***, ***CNAME***, ***MX***가 있습니다. 이 외에도 여러가지가 있지만, 다른 ***RECORD***들은 특별한 기능을 위해서 사용하므로, 해당 기능 문서에서 다루게 될 것입니다.


### 5.2.2.3 SOA 영역

zone database의 시작은 항상 ***SOA*** RECORD로 시작을 합니다. SOA 레코드는 해당 도메인, ***annyung.oops.org***에 대해 네임서버가 인증(authoritative)된 자료를 갖고 있음을 의미하며, 자료가 최적의 상태로 유지, 관리될 수 있도록 합니다.