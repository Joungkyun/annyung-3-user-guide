# Chapter 5.5 DNSSEC 구성

***DNSSEC(DNS Security Extensions)***는 도메인 정보의 위/변조를 방지하기 위하여 기존의 DNS 시스템 표준에 대해 공개키 암호화 방식의 전자서명 메커니즘을 적용한 확장표준 입니다.

***DNSSEC***에 대한 이해는 [KISA에서 제공하는 문서](http://krnic.or.kr/jsp/resources/dns/dnssecInfo/dnssecInfo.jsp)를 참고 하시기 바랍니다.


## 5.5.1 Zone key 생성

Zone key 생성은 ***KSK(Key signing key)***와 ***ZSK(Zone signing key)*** 두가지를 생성해야 합니다. 일단, ***KSK***와 ***ZSK***는 ***/var/named/etc/pki/dnssec-keys***에 생성을 하도록 합니다.

생성된 키는 ***Z[zone_name]+[algorithm]+[id].key*** 의 형식으로 생성이 됩니다.

### 5.5.1.1 KSK(Key signing key) 생성

***KSK***는 ***ZSK(Zone signing key)***를 서명하는데 사용을 합니다.

생성은 아래 과정으로 진행을 합니다. 주의할 것은 도메인 마지막에 ***dot(.)***를 붙여 이름 확장이 되지 않도록 해야 합니다. (이름 확장 부분은 정확히 확인이 된 사항은 아닙니다. 일단은 안전하게 ***dot(.)***을 붙여 주는 것을 권장 합니다.)

```bash
[root@ns ~]$ cd /var/named/etc/pki/dnssec-keys
[root@ns dnssec-keys]$ dnssec-keygen -3 -r /dev/urandom -b 2048 -n ZONE -f KSK domain.org.
Generating key pair.....................+++ ...................................+++
Kdomain.org.+007+37828
[root@ns dnssec-keys]$ ls
Kdomain.org.+007+37828.key   Kdomain.org.+007+37828.private
[root@ns dnssec-keys]$
```

옵션을 다음과 같습니다.

* ***-3*** : ***DNSSEC*** 키를 생성하는데 ***NSEC3***가 가능한 알고리즘을 사용. 기본값으로는 ***NSEC3RSASHA1***를 사용
* ***-r*** : key 생성을 위한 random device를 지정
* ***-b*** : key size (bit)
* ***-n*** : Name Type (default : Zone)
* ***-f*** : key flag

자세한 옵션은 ***dnssec-keygen***의 man page를 확인 하십시오.

### 5.5.1.2 ZSK(Zone signing key) 생성

***ZSK***는 zone의 모든 ***RR***을 서명하는데 사용 합니다.

생성은 아래 과정으로 진행을 합니다. 주의할 것은 도메인 마지막에 ***dot(.)***를 붙여 이름 확장이 되지 않도록 해야 합니다.

```bash
[root@ns ~]$ cd /var/named/etc/pki/dnssec-keys
[root@ns dnssec-keys]$ dnssec-keygen -3 -r /dev/urandom -b 2048 -n ZONE -I 2018021200000000 -D 2018031200000000 domain.org.
Generating key pair...........+++ .....................+++
Kdomain.org.+007+28385
[root@ns dnssec-keys]$
```

***KSK***는 영구 key이지만, ***ZSK***는 기본으로 생성 시점에서 3개월 후에 만료가 됩니다. 그러므로 몇년치 ***ZSK***를 미리 생성해 놓고, ***auto-dnssec*** 설정을 이용하여 자동 갱신 시키는 방법도 있지만, 여기서는 자동 갱신에 대해서는 언급을 하지 않겠습니다. 자동 갱신에 대해서는 아래 ***5.5.7 참고 문서*** 의 URL을 참고 하십시오.



여기서는 inactive를 12개월, expire를 13개월로 설정을 했습니다.

* -P : publication date (default: now) 
* -A : activation date (default: now)
* -I : Inactivation date 
* -D : deletion date

지정한 시간의 Timezone은 ***UTC***이며, ***YYYYMMDDHHMMSS*** 형식을 사용 합니다.

***참고:***
***auto-dnssec***를 사용하려면 key file을 ***bind*** process가 읽을 수 있는 권한이 있어야 합니다. 그러므로 생성된 key file들을 ***named*** user권한으로 변경 하십시오.

```
[root@ns ~]$ cd /var/named/etc/pki/dnssec-keys
[root@ns dnssec-keys]$ chown named:named *.key *.private
```

## 5.5.2 Zone sign

***KSK***와 ***ZSK*** 생성이 완료가 되었다면 이제 zone file을 서명 하도록 합니다.

```bash
[root@ns dnssec-keys]$ cd /var/named/zone
[root@ns zone]$ dnssec-signzone -S -K /var/named/etc/pki/dnssec-keys -3 67136a -e 20180212000000 -o domain.org. domain.org.zone
Fetching KSK 9870/NSEC3RSASHA1 from key repository.
Fetching ZSK 28385/NSEC3RSASHA1 from key repository.
Verifying the zone using the following algorithms: NSEC3RSASHA1.
Zone fully signed:
Algorithm: NSEC3RSASHA1: KSKs: 1 active, 0 stand-by, 0 revoked
                         ZSKs: 1 active, 0 stand-by, 0 revoked
domain.org.zone.signed
```

주의할 것은 ***-S*** (smart key) 옵션을 사용하지 않으면, 생성된 ***KSK***와 ***ZSK***를 zone file안에 include 해 줘야 합니다. ***-S*** 옵션을 사용하면 include 하지 않아도 상관이 없습니다. 여기서는 ***-S*** 옵션을 사용했기 때문에 zone file에 key를 include 하지 않습니다.

제대로 생성이 되었다면, ***domain.org.zone.signed*** 와 ***dsset-domain.org.*** 파일이 생성이 됩니다. 



## 5.5.3 siged zone 파일 등록

***/var/named/etc/named.user.conf*** 에서 정의된 zone 영역에서 zone file을 새로 생성된 zone file로 변경한 다음, ***bind***를 재시작 합니다.

```
zone "domain.org" IN {
    type master;
    //file "domain.org.zone";
    file "domain.org.zone.signed";
    allow-update { none; };
};
```

expire 자동 갱신을 위해서는 별도의 옵션이 더 필요하나, 여기서는 자동 갱신을 안하기 때문에, 위와 같이 ***file*** 옵션 값만 변경해 주면 됩니다. 자동 갱신에 대해서는 아래 ***5.5.7 참고 문서*** 의 URL을 참고 하십시오.

다음의 명령으로 ***bind*** process를 갱신 합니다.

```bash
[root@ns zone]$ rndc reload
or
[root@ns zone]$ service named reload
```

## 5.5.4  Slave 전송

***Slave*** 구성이 되어 있다면, ***Slave***에서는 별도로 할 일이 없고, ***Master***에서 zone sign을 하기 전에 ***SOA*** 필드의 ***Searial*** 값만 증가 시킨 후에 sign을 하면 알아서 전송이 됩니다.

## 5.5.5 상위 registrar에 DS record 등록

이 부분이 가장 난관 입니다. 도메인 registrar에서 지원을 해 줘야 하는데, 필자가 보유한 domain registra가 지원하지 않아 확인을 하지 못했습니다.

일단, org extensiondp 대한 DNSSEC를 지원하는 registrar의 목록은 다음 URL에서 확인할 수 있습니다.

http://pir.org/products/find-a-registrar/?order=field_dnssec_value&sort=desc

___yes*___ 와 같이 yes옆에 astrik가 붙어 있으면 ***DNSSEC***를 지원하는 registrar 입니다.

이 작업이 되지 않으면, 상위 registra와의 deletation이 ***SECURE*** 상태가 되지 못합니다. 많은 문서들이 이 부분에 대한 내용이 누락되어 있습니다.

## 5.5.6 DNSSEC 검증

http://krnic2014.websrv.co.kr/jsp/resources/dns/dnssecInfo/dnssecTool.jsp
http://dnsviz.net/
http://dnssec-debugger.verisignlabs.com/

### 5.5.7 참고 문서
https://www.dragonsreach.it/2013/11/13/configuring-dnssec-personal-domain/
https://sites.google.com/site/dnsportalkorea/home
http://krnic.or.kr/jsp/resources/dns/dnssecInfo/dnssecBind.jsp
