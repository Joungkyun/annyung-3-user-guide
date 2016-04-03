# NIS intergrate

이 장에서는 NIS를 이용한 인증 통합에 대하여 기술을 합니다.

> 목차
> 1. NIS
> 2. NIS 구성 시 고려 사항
> 1. NIS server 설정
> 2. NIS slave 설정
> 3. NIS client 설정

##1. NIS(Network Information Service)


네트워크 정보 서비스(Network Information Service, NIS)는 ~~썬 마이크로시스템즈~~(현 Oracle사에 인수됨)의 클라이언트 서버 디렉터리 서비스 프로토콜이며, 컴퓨터 네트워크 위의 컴퓨터들 사이에 있는 사용자와 호스트 이름과 같은 시스템 구성 데이터를 여러 곳에 제공합니다.(출처 [wikipedia](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%EC%A0%95%EB%B3%B4_%EC%84%9C%EB%B9%84%EC%8A%A4))

##2. NIS 구성 시 고려 사항

***NIS***는 secure protocol이 없습니다. 이는 네트워크 상에 passwd list가 평문(plain text)로 전송이 되어 쉽게 sniffing이 될 수 있다는 의미입니다. 그러므로, NIS 구성은 매우 제한된 network에서 구성을 해야 합니다.

1. Private network 상에서 구성할 것
2. 서로 다른 Network 구간에서 연동이 필요할 경우, VPN tunnel을 이용할 것
3. data를 text 파일로 관리해야 하기 때문에 많은 account를 관리해야 할 경우 권장하지 않음.
4. Multi master 구성이 안됨


##3. NIS server 설정

###3.1 NIS server package 설치

```bash
[root@an3 ~]$ yum install ypserv rpcbind genpasswd
```

###3.2 NIS domain 설정

NIS domain이라는 것은 NIS database 이름 정도라고 생각을 하면 됩니다. 대충 사용하시는 도메인을 이용하시면 됩니다. 예를 들어서. ***oops.org*** 또는 ***OOPS-NIS***, 이것도 귀찮으면 ***OOPS***와 같이 지정을 하면 됩니다.

```bash
[root@an3 ~]$ ypdomainname OOPS-NIS
[root@an3 ~]$ echo "NISDOMAIN=\"OOPS-NIS\"" >> /etc/sysconfig/network
```

###3.3 기본 설정

안녕 리눅스의 NIS 설정 파일은 */var/yp* 디렉토리에 존재 합니다.

보통 NIS 설정을 할때 시스템 상의 */etc/passwd*와 */etc/group*을 이용하여 database를 만드는 것을 권장하는데, 여기서는 이 파일들을 사용하지 않고 별도의 파일로 관리하도록 기술 합니다.

###3.3.1 passwd/group list 파일 준비

```bash
[root@an3 ~]$ mkdir -p /var/yp/etc
```




##4. NIS slave 설정

##5. NIS client 설정

