# GeoDNS 설정

> 목차
>
> 5.6.1 GeoDNS 란?
>
> 5.6.2 GeoDNS 설정

## 5.6.1 GeoDNS 란?

> **주의:**
>
> 안녕 리눅스 3.5 \(CentOS 7.7\) 부터는 GeoDNS 방식이 기존의 Google GeoDNS 방식에서 ISC GeoDNS 방식으로 변경 되었습니다. bind 버전이 9.11 이상 이라면 [https://kb.isc.org/docs/aa-01149](https://kb.isc.org/docs/aa-01149) 를 참고 하십시오. 현재 이 문서 는 deprecated 되어진 Google GeoDNS 방식을 기술하고 있습니다.

GeoDNS 기능은, GeoIP의 database를 이용하여 조건에 따라 다른 응답을 하는 기능을 의미 합니다.

예를 들어, 한국에서 접근을 했을 때에는 한국에 있는 서버의 IP를 반환고, 한국이 아닌 곳에서 접근을 했을 경우에는 미국에 있는 서버의 IP를 반환 하도록 설정을 할 수 있습니다.

이 기능을 제대로 사용하기 위해서는 주기적은 GeoIP database 갱신이 필요 합니다.

사용할 수 있는 GeoIP database는 다음과 같습니다.

* Country
* City
* Region
* ISP
* Organization
* AS number
* Netspeed
* Domain
* Country IPv6

RHEL 7.3 또는 CentOS 7.3 부터 bind에 GeoDNS 기능을 제공합니다. 하지만, 안녕 리눅스의 경우, RHEL 또는 CentOS 보다 먼저 이 기능을 제공하기 시작했으며, 제공하는 방식이 다릅니다. 그래서 RHEL 7.3 기반인 안녕 리눅스의 bind에서는 안녕 리눅스 2/3의 하위 호환성을 위하여 RHEL 7.3의 방식과는 다른 방식으로 제공합니다.

자세한 사용법에 대해서는 [https://code.google.com/archive/p/bind-geoip/wikis/UsageGuide.wiki](https://code.google.com/archive/p/bind-geoip/wikis/UsageGuide.wiki) 문서를 참고 하시고, 여기서는 국가별 제어에 대한 예제만 다루도록 합니다.

## 5.6.2 GeoDNS 설정

GeoDNS 기능은 _**bind**_의 _**view**_ 기능을 이용하여 제공 합니다. _**view**_ 기능은 client의 특정 상태에 따라 다른 응답을 할 수 있도록 하는 기능 입니다.

일단 view 작업은 _**/var/named/etc/named.user.zones**_ 에서 하도록 합니다. 일단 예제 부터..

```text
view "US" {
    match-clients {
        geoip_countryDB_country_US;
    };

    zone "cdn.mydomain.com" IN {
        type master;
        file "US/cdn.mydoman.com.zone";
    }
};

view "EASTASIA" {
    match-clients {
        geoip_countryDB_country_KR;
        geoip_countryDB_country_JP;
        geoip_countryDB_country_CN;
    };

    zone "cdn.mydomain.com" IN {
        type master;
        file "KR/cdn.mydoman.com.zone";
    };
};

view "default" {
    zone "cdn.mydomain.com" IN {
        type master;
        file "DEF/cdn.mydoman.com.zone";
    }
}
```

_**match-clients**_의 조건에 따라서 설정된 zone 파일을 참조하게 됩니다. _**match-clients**_ 조건은 다음의 형식으로 GeoIP country를 사용합니다.

```text
geoip_<DBTYPE>DB_<FIELD>_<VALUE>
```

위의 예제로 보자면 다음과 같습니다.

```text
DBTYPE : geoip
FIELD:   country
VALUE:   국가코드 (ISO 3166-1 alpha-2)
```

GeoIP database의 country 필드의 값에 따라 다르게 적용을 하라는 의미입니다. 국가 코드는 GeoIP의 country fileld 값인 _**ISO3166-1**_ 두자리 알파뱃을 이용 합니다.

