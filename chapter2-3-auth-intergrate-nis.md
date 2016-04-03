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

NIS에서 관리할 passwd/group 파일은 */etc/passwd*와 */etc/group* 과 동일한 format 을 사용합니다.

```bash
[root@an3 ~]$ mkdir -p /var/yp/etc
[root@an3 ~]$ getpasswd -m md5
New Password:
Retype New Password:
$1$p93nsknZ$WRwO/47kxt7dheszvNliy.
[root@an3 ~]$ echo "USERID:$1$p93nsknZ$WRwO/47kxt7dheszvNliy.:10000:10000:REAL USER NAME:/home/admin/oops:/bin/bash" >> /var/yp/etc/passwd
[[root@an3 ~]$ echo "nisusers:x:10000:" >> /var/yp/etc/group
```

연동할 시스템 중에 오래된 OS가 있다면 암호는 ***md5*** 방식으로 선택 합니다. 최신의 OS들로만 구성되어 있다면 sha512를 선택하는 것을 권장합니다.

안녕 리눅스에서 제공하는 ***genpasswd*** 프로그램은 md5, sha256, sha512 방식의 암호 문자열을 생성할 수 있습니다.

###3.3.2 /var/yp/Makefile 및 /etc/sysconfig/yppasswdd 설정

***/var/yp/Makefile*** 중에서 다음의 설정들을 수정합니다.

```bash
# slave NIS를 구성할 것이라면 값을 true로 변경 합니다. 기본값은 false 입니다.
# slave NIS를 구성할 것이 아니라면 false로 나두십시오.
NOPUSH=true

# 인증 통합시에 system uid/gid와 충돌할 경우가 발생할 수 있습니다. 그러므로
# 충분한 값을 주도록 합니다. 대략 10000번 이상대를 사용하면 거의 충돌할 일이
# 없습니다.
MINUID=10000
MINGID=10000

# system의 passwd/group 을 사용하지 않을 것이기 때문에, false 로 설정 합니다.
MERGE_PASSWD=false
MERGE_GROUP=false

# system의 passswd/group을 사용하지 않을 것이기 때문에, 위에서 작업한 디렉토리로
# passwd/group file이 위치한 경로를 변경해 줍니다.
YPPWDDIR = /var/yp/etc

# If you don't want some of these maps built, feel free to comment
# them out from this list.
# passwd/group 만 다루기 때문에 생성할 DB를 제한 합니다. (제한하지 않으면 설정
# 누락으로 DB 생성 실패 되는 경우가 있습니다.) 여기서는 기존의 설정에서
# ***netid*** 만 제거 하였습니다.
all:  passwd group hosts rpc services protocols mail \
    # netgrp shadow publickey networks ethers bootparams printcap netid \
    # amd.home auto.master auto.home auto.local passwd.adjunct \
    # timezone locale netmasks
```

다음, ***/etc/sysconfig/yppasswd*** 를 다음과 같이 수정 합니다.

```bash
[root@an3 ~]$ cat /etc/sysconfig/yppasswd
# The passwd and shadow files are located under the specified
# directory path. rpc.yppasswdd will use these files, not /etc/passwd
# and /etc/shadow.
ETCDIR=/var/yp/etc

  .. 중략 ..

# Additional arguments passed to yppasswd
# 방화벽 설정을 위해 port를 static 하게 설정 합니다.
YPPASSWDD_ARGS="--port 836"
[root@an3 ~]$
```


###3.3.3 보안 설정

NIS 질의를 할 수 있는 네트워크 대역을 제한 합니다. 형식은 ***NETMASK NETWORK*** 형식으로 설정 합니다. 다음의 설정은 127.0.0.0/8 과 192.168.0.0/24 네트워크에서 NIS 질의에 응답하도록 설정한 것입니다.

```bash
[root@an3 ~]$ cat > /var/yp/securenets <<EOF
255.0.0.0 127.0.0.0
255.255.255.0 192.168.0.0
EOF
[root@an3 ~]$
```

방화벽을 사용한다면 ypserv와 ypxfrd, yppasswdd의 포트를 고정 시키고 port를 열어주어야 합니다.

```bash
[root@an3 ~]$ cat >> /etc/sysconfig/network <<EOF
YPSERV_ARGS="-p 834"
YPXFRD_ARGS="-p 835"
EOF
[root@an3 ~]$ 
[root@an3 ~]$ cat /etc/oops-firewall/filter.conf
  ... 상략 ...
# RPCBIND tcp/udp 111
# YPSERV tcp/udp 834
# YPXFRD tcp/udp 835
TCP_HOSTPERPORT = 192.168.0.0/24:111 192.168.0.0/24:834-836
UDP_HOSTPERPORT = 192.168.0.0/24:111 192.168.0.0/24:834-836
  ... 하략 ...
```

###3.3.3.4 Daemon 실행

```bash
[root@an3 ~]$ service rpcbind start
[root@an3 ~]$ service ypxfrd start
[root@an3 ~]$ service yppasswdd start
[root@an3 ~]$
```





##4. NIS slave 설정

##5. NIS client 설정

