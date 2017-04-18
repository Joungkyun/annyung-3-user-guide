## Chapter 5.8 IDN

>목차
5.8.1 punycode
5.8.2 IDN 환경
5.8.3 한글 도메인 설정
  * 5.8.3.1 domain zone 설정
  * 5.8.3.2 zone file 설정

IDN(Internationalized Domain Name)은 흔히 다국어 도메인이라고 말을 하기도 합니다.


### 5.8.1 punycode

IDN은 1998년 부터 논의가 시작이 되었으며, 처음 IDN 서비스를 시작한 verysign은 RACECODE 를 이용하여 COM/NET 도메인에 IDN을 적용하여 서비스 하기 시작했으나, 2002년 [punycode](https://ko.wikipedia.org/wiki/%ED%93%A8%EB%8B%88%EC%BD%94%EB%93%9C)가 표준안으로 채택이 되고 나서 2004년 경 부터는 모든 domain extension들이 punycode를 사용하기 시작했으며, 현재는 punycode만 사용이 되어 집니다.

punycode는 domain에 사용할 수 있는 문자가 아닐 경우에는,  다음과 같이 ***xn--*** prefix를 이용하여 변환을 합니다.

```
안녕.com =>       xn--o70b819a.com
안녕-linux.com => xn---linux-2x2xk18i.com
```

IDN을 punycode로 변환을 하기 위해서는 보통 웹상에서 "punycode 변환"을 검색어로 검색을 하면 많은 페이지를 볼 수 있습니다.

안녕 리눅스의 경우, whois package를 설치할 경우, command line interface에서 사용을 할 수 있는 ***punyconv*** 실행 파일이 설치가 됩니다.

```
[root@an3 ~]$ punyconv 안녕.com
xn--o70b819a.com
[root@an3 ~]$
```

### 5.8.2 IDN 환경

안녕 리눅스에서의 지원하는 패키지들의 경우에는 대부분 내부에서 punycode를 지원하도록 패치가 되어 있습니다. ***ssh***, ***ping***, ***traceroute***, host 등등의 network 도구에서는 IDN을 직접 사용하더라도 내부에서 알아서 punycode로 변환을 하도록 되어 있습니다.

```
[root@main ~]$ ping 한글.com
PING 한글.com (202.31.187.154) 56(84) bytes of data.
64 bytes from 202.31.187.154 (202.31.187.154): icmp_seq=1 ttl=52 time=7.79 ms
64 bytes from 202.31.187.154 (202.31.187.154): icmp_seq=2 ttl=52 time=8.22 ms
64 bytes from 202.31.187.154 (202.31.187.154): icmp_seq=3 ttl=52 time=7.83 ms
```

### 5.8.3 한글 도메인 설정

#### 5.8.3.1 zone
기본적으로, ***bind***에서 IDN을 사용하기 위해서는 punycode를 사용해야 합니다. 예를 들어 ***한글.com***을 설정 한다고 가정 하면 다음과 같습니다.

```
zone "xn--bj0bj06e.com" IN {
    type master;
    file "zone/한글.com.zone";
    allow-update { none; };
};
```

안녕 리눅스에서는 다음과 같이 punycode 변환 없이 사용을 할 수 있습니다.

```
zone "한글.com" IN {
    type master;
    file "zone/한글.com.zone";
    allow-update { none; };
};
```

