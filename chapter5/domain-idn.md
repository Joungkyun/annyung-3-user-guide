# IDN

> 목차 5.8.1 punycode 5.8.2 IDN 환경 5.8.3 한글 도메인 설정
>
> * 5.8.3.1 IDN zone 설정
> * 5.8.3.2 zone file에서의 IDN 사용

IDN\(Internationalized Domain Name\)은 흔히 다국어 도메인이라고 말을 하기도 합니다.

## 5.8.1 punycode

IDN은 1998년 부터 논의가 시작이 되었으며, 처음 IDN 서비스를 시작한 verysign은 RACECODE 를 이용하여 COM/NET 도메인에 IDN을 적용하여 서비스 하기 시작했으나, 2002년 [punycode](https://ko.wikipedia.org/wiki/%ED%93%A8%EB%8B%88%EC%BD%94%EB%93%9C)가 표준안으로 채택이 되고 나서 2004년 경 부터는 모든 domain extension들이 punycode를 사용하기 시작했으며, 현재는 punycode만 사용이 되어 집니다.

punycode는 domain에 사용할 수 있는 문자가 아닐 경우에는, 다음과 같이 _**xn--**_ prefix를 이용하여 변환을 합니다.

```text
안녕.com =>       xn--o70b819a.com
안녕-linux.com => xn---linux-2x2xk18i.com
```

IDN을 punycode로 변환을 하기 위해서는 보통 웹상에서 "punycode 변환"을 검색어로 검색을 하면 많은 페이지를 볼 수 있습니다.

안녕 리눅스의 경우, whois package를 설치할 경우, command line interface에서 사용을 할 수 있는 _**punyconv**_ 실행 파일이 설치가 됩니다.

```text
[root@an3 ~]$ punyconv 안녕.com
xn--o70b819a.com
[root@an3 ~]$
```

## 5.8.2 IDN 환경

안녕 리눅스에서의 지원하는 패키지들의 경우에는 대부분 내부에서 punycode를 지원하도록 패치가 되어 있습니다. _**ssh**_, _**ping**_, _**traceroute**_, host 등등의 network 도구에서는 IDN을 직접 사용하더라도 내부에서 알아서 punycode로 변환을 하도록 되어 있습니다.

```text
[root@main ~]$ ping 한글.com
PING 한글.com (202.31.187.154) 56(84) bytes of data.
64 bytes from 202.31.187.154 (202.31.187.154): icmp_seq=1 ttl=52 time=7.79 ms
64 bytes from 202.31.187.154 (202.31.187.154): icmp_seq=2 ttl=52 time=8.22 ms
64 bytes from 202.31.187.154 (202.31.187.154): icmp_seq=3 ttl=52 time=7.83 ms
```

좀더 요약하자면, shell의 언어 환경이 _**UTF-8**_ 이라면, punycode를 신경쓸 필요 없다는 의미 입니다.

## 5.8.3 한글 도메인 설정

### 5.8.3.1 IDN zone 설정

기본적으로, _**bind**_에서 IDN을 사용하기 위해서는 punycode를 사용해야 합니다. 예를 들어 _**한글.com**_을 설정 한다고 가정 하면 다음과 같습니다.

```text
zone "xn--bj0bj06e.com" IN {
    type master;
    file "zone/한글.com.zone";
    allow-update { none; };
};
```

안녕 리눅스에서는 다음과 같이 punycode 변환 없이 사용을 할 수 있습니다.

```text
zone "한글.com" IN {
    type master;
    file "zone/한글.com.zone";
    allow-update { none; };
};
```

IDN을 작성할 때, zone 파일의 문자셋이 _**UTF-8**_일 경우에는 모든 IDN을 punycode로 변환 없이 사용이 가능하며, _**UTF-8**_이 아닐 경우에는 _**EUC-KR**_ 과 _**EUC-JP**_만 지원을 합니다.

또한, _**EUC-KR**_ 또는 _**EUC-JP**_를 사용하기 위해서는 _**CH+문자셋.도메인**_의 형식을 이용해야 합니다.

```text
zone "CH+EUC-KR.한글.com" IN {
    type master;
    file "zone/한글.com.zone";
    allow-update { none; };
};
```

### 5.8.3.2 zone file에서의 IDN 사용

zone 설정과 마찬 가지로, _**UTF-8**_ 환경이라면, 아무런 제약 없이 _**IDN**_을 직접 사용할 수 있습니다.

```text
자음         IN  A   10.0.0.1
```

이렇게 설정을 하면, 자음.한글.com 의 A record가 10.0.0.1로 설정이 되게 됩니다.

zone 설정과 마찬가지로, _**UTF-8**_ 환경이 아니라면, _**EUC-KR**_ 및 _**EUC-JP**_만을 지원 하며, IDN 표기를 _**CH+문자셋.도메인**_의 형식을 이용해야 합니다.

```text
CH+EUC-KR.자음         IN  A   10.0.0.1
```

단, 혼용을 하지 않는 다는 가정하에서, ZONE 파일 제일 처음에 _**@CHARSET**_ 지시자를 이용할 수 있습니다.

```text
@CHARSET "EUC-KR"
$TTL 60
@               IN  SOA ns.oops.org. admin.oops.org.    (
                2017021205  ; Serial
                10800       ; Refresh
                3600        ; Retry
                604800      ; Expire
                60          ; TTL (1day)
                )

자음         IN  A   10.0.0.1
```

위와 같이 사용이 가능 합니다.

