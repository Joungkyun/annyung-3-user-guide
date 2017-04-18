## Chapter 5.8 IDN

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



