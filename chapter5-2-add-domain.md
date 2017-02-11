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
                IN  NS      ns2.oops.org.
                IN  MX 10   mail
                IN  A       111.112.113.1

www             IN  CNAME   @
ftp             IN  CNAME   @
mail            IN  A       111.112.113.114
                IN  TXT     "v=spf1 include:annyung.oops.org ~all"

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

### 5.2.2.2 도메인 이름 확장

도메인 이름 확장은 zone 파일 설정에서 아주 중요한 부분 입니다. 대부분의 DNS 설정 오류가 도메인 이름 확장을 잘못 사용하여 발생을 하게 됩니다.

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

***DOMAIN*** 필드가 생략이 되었을 경우에는, 바로 위의 record의 DOMAIN을 상속 받습니다.

```
www      IN    A     1.1.1.1
         IN    A     1.1.1.2
```

위의 설정은 다음과 동일 합니다.

```
www      IN    A     1.1.1.1
www      IN    A     1.1.1.2
```

### 5.2.2.4 SOA record 영역

zone database의 시작은 항상 ***SOA*** RECORD로 시작을 합니다. SOA 레코드는 해당 도메인, ***annyung.oops.org***에 대해 네임서버가 인증(authoritative)된 자료를 갖고 있음을 의미하며, 자료가 최적의 상태로 유지, 관리될 수 있도록 합니다.

***SOA*** record 설정의 형식은 다음과 같습니다.

```
[ORIGN] [CLASS] [RECORD] [PRIMARY DNS] [E-MAIL]        ([SERIAL]   [REFRESH] [RETRY] [EXPIRE] [TTL])
@       IN      SOA      ns.oops.org.  admin.oops.org. (2017011500 10800     3600    604800   86400)
```

***PRIMARY DNS***는 origin에 대한 primary name server 이름을 지정 합니다.
***E-MAIL***은 이 도메인을 관리하는 name server 관리자의 메일 주소를 지정 합니다. 단 메일 주소의 ***@***을 ***dot(.)***으로 표기하는 것을 주의해야 합니다. 해당 도메인의 contact point로 사용이 되며, 도메인에 문제가 있을 경우에 이에 대한 리포팅 대상으로 사용되기 됩니다. 또한 Namespace를 쫒으며 도메인 오류를 점검하는 lamers와 같은 도구들이 문제를 검출 하였을 경우, 이 메일 주소로 통지를 하게 됩니다.

다음 괄호로 둘러싸인 부분엔 ***Serial***, ***Refresh***, ***Retry***, ***Expire***, ***Minimum*** 5개의 시간을 지정하는 필드가 정의 됩니다. ***TTL***을 제외한 처음 4개 필드는 Secondary 또는 Slave 네임서버를 제어하기 위한 값 d입니다. 기본 단위는 '초'이고, 단위기호 M(Minute), H(Hour), D(Day), W(Week)를 붙여 30M, 8H, 2D, 1W와 같이 사용할 수 있습니다.

* ***Serial*** :
  Secondary 또는 Slave가 zone 파일의 수정 여부를 알 수 있도록 하기 위하여 사용합니다. Slave DNS 설정이 되어 있을 경우, ***Master*** 서버는 reload 또는 restart signal을 받을 경우, 각 zone 파일의 ***serial*** 값을 ***Slave*** 서버로 전송하게 됩니다. ***serial*** 값을 전송받은 ***Slave*** 서버는 백업본의 ***serial***이 전송받은 값 보다 작을 경우 zone 파일을 재전송 받게 됩니다. 즉, ***Master***에서 zone 파일이 변경이 되었더라도 ***serial***값이 변경이 되지 않는다면 ***Slave*** 서버들이 데이터는 갱신이 되지 않는다는 의미 입니다. 그러므로 ***Slave*** DNS가 없더라도 항상 zone file을 변경한 후에는 ***serial*** 값을 갱신 시키는 습관을 들이는 것을 권장 합니다.
  
  ***serial***의 표기법은 최근의 ***bind***에서는 증가하는 임의의 숫자는 deprecated 되어 사용을 지양하고 있으며, 최종 수정일을 ***YYYYMMDDNN***의 형식으로 표기 합니다. ***YYYYMMDDNN*** 연도 표기법은 4294년까지 가능 합니다.

* ***Refresh*** :
  ***Slave*** DNS가 ***Master***의 zone database 수정 여부를 검사하는 주기 입니다. 이 기능은 ***Master***의 notify가 실패를 하였을 경우를 대비하여 ***Slave***측에서 동기화를 어긋나지 않도록 하기 위한 기능 입니다. 

* ***Retry*** :
  ***Slave***측에서, ***Master***와 연결이 되지 않을 경우, 재 시도 주기 입니다. ***Refresh*** 주기보다 클 경우에는 의미가 없습니다.

* ***Expire*** :
  ***Slave***가 ***expire***로 지정된 시간동안 동기화가 이루어 지지 못할 경우, 오래된 백업 카피의 자료를 폐기 합니다. 이 값을 너무 작게 책정하는 것은 좋지 않고 보통 1~2주 정도로 설정을 하면 됩니다.것은 좋지 않다. 보통 1W - 2W(1209600)로 설정한다.

* ***TTL***

  caching name server가 본 zone database 자료를 가지고 있을 때, 이 database에 대한 유효 기간을 설정 합니다. TTL 값이 명시되지 않은 record들은 이 값을 기본 TTL로 설정이 됩니다. ***0***은 caching을 하지 말라는 의미입니다.  Bind 8에서 ***$TTL*** keyword가 추가된 이후로는 이 값을 이용하여 TTL 설정을 하기 보다는 ***$TTL*** keyword를 이용해서 기본 TTL 값을 설정하는 것이 근래의 추세 입니다.  
  
  이 값이 너무 길게 지정이 될 경우에는 변경된 record가 다른 DNS로 전파되는데 오래 걸리는 단점이 있습니다. 그러므로 record 변경 전에 TTL을 감안하여 미리 TTL을 줄여 놓고 작업을 해야할 수도 있습니다.
  
### 5.2.2.5 NS record

***NS*** record를 이용하여 해당 도메인의 name server를 나타냅니다.

```
@          IN  NS  ns.oops.org.
           IN  NS  ns2.oops.org.
```

***origin(annyung.oops.org)***는 ***ns.oops.org***와 ***ns2.oops.org***에서 관리 되어진다고 announce 하게 됩니다.

또한, ***NS*** record는 서브 도메인의 권한을 다른 DNS에 위임할 경우에도 사용할 수 있습니다.

```
sub        IN  NS ns.sub.annyung.oops.org.
           IN  A  111.112.113.120
```

위의 설정은 ****.sub.annyung.oops.org*** 의 권한을 ***ns.sub.annyung.oops.org***에 위임한다는 설정입니다. 여기서 위임한 네임 서버(ns.sub.annyung.oops.org)의 ***A*** record(IP 주소)를 ***glue record***라고 합니다. ***glue record***를 언급하는 이유는, 위임을 하는 DNS가 신규로 만들어 지는 DNS일 경우에는 위와 같이 ***glue record***를 설정해야 하지만, 기존의 운영되고 있는 DNS로 위임을 하는 경우에는 ***glue record***를 설정하지 않는 것이 좋습니다.

예를 들어, 기존에 운영이 되고 있는 ***ns.dns.com(10.10.10.10)*** 서버로 위임을 할 경우, 아래와 같이 ***glue record***를 설정하지 말아야 합니다.

* ***잘못된 예:*** 이미 존재하는 DNS에 대한 이름을 또 만들지 않아야 합니다!
  ```
  sub        IN  NS ns.sub.annyung.oops.org.
             IN  A  10.10.10.10
  ```

* ***올바른 예:***
  ```
  sub        IN  NS ns.dns.com.
  ```

### 5.2.2.6 A(Address) & CNAME(Canonical Name)

***A*** record와 ***CNAME*** record는 도메인에 대한 IP 주소나 별칭을 설정 하는데 사용을 합니다. IP 주소를 지정할 경우에는 ***A*** record를 사용하며, 별칭을 설정할 때는 ***CNAME*** record를 사용 합니다.

```
sub        IN  A     1.1.1.1
sub1       IN  CNAME sub
```

위의 설정은 다음을 의미합니다.

* ***sub.annyung.oops.org***의 IP 주소는 1.1.1.1
* ***sub1.annyung.oops.org***는 ***sub.annyung.oops.org***와 동일함.

하나의 도메인에 대해서 여러개의 ***A*** record를 부여하는 것을 ***DNS RR(Round Robin)*** 이라고 합니다.

```
sub        IN  A     1.1.1.1
           IN  A     1.1.1.2
           IN  A     1.1.1.3
```

위와 같이 설정을 하면, 요청이 올 때 마다 차례대로 하나씩 응답을 하게 됩니다. (서버 입장에서 요청온 순서대로 차례대로 이기 때문에 client 입장에서는 동일한 IP를 연속으로 받을 수도 있습니다.) ***DNS를 이용한 서버 부하 분산***을 할 경우에 이 설정을 이용할 수 있습니다. 하지만, 지정된 서버의 fail over를 할 수 없기 때문에 대부분 서버 부하 분산은 DNS를 이용하기 보다는 ***L4***나 ***L7*** 장비 또는 software를 이용하는 것이 바람직 합니다.

***A*** record를 이용하여 ***DNS RR***을 처리 하듯이 ***CNAME*** record를 이용하여 할 수도 있습니다. 하지만, ***CNAME***을 이용한 ***DNS RR***은 안녕 리눅스에서 제공하는 ***bind*** 또는 ***bind*** v4 에서나 가능 합니다. 사실 multiple ***CNAME***은 DNS의 규약 위반 입니다. 그러므로 ***bind*** v8 이후에는 제거가 되었습니다. 하지만 간혹 유용하게 쓰일 경우가 있어 안녕 리눅스의 ***bind***에서는 multiple ***CNAME***을 지원 합니다. (하지만 될 수 있으면 사용하지 마십시오!)















