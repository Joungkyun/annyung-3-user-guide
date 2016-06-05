# NIS 인증 통합

이 장에서는 NIS를 이용한 인증 통합에 대하여 기술을 합니다.

> 목차
> 1. NIS
> 2. NIS 구성 시 고려 사항
> 3. NIS server 설정
> 4. NIS slave 설정
> 5. NIS client 설정

##1. NIS(Network Information Service)


네트워크 정보 서비스(Network Information Service, NIS)는 ~~썬 마이크로시스템즈~~(현 Oracle사에 인수됨)의 클라이언트 서버 디렉터리 서비스 프로토콜이며, 컴퓨터 네트워크 위의 컴퓨터들 사이에 있는 사용자와 호스트 이름과 같은 시스템 구성 데이터를 여러 곳에 제공합니다.(출처 [wikipedia](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%EC%A0%95%EB%B3%B4_%EC%84%9C%EB%B9%84%EC%8A%A4))

##2. NIS 구성 시 고려 사항

***NIS***는 secure protocol이 없습니다. 이는 네트워크 상에 passwd list가 평문(plain text)로 전송이 되어 쉽게 sniffing이 될 수 있다는 의미입니다. 그러므로, NIS 구성은 매우 제한된 network에서 구성을 해야 합니다.

1. Private network 상에서 구성할 것
2. 서로 다른 Network 구간에서 연동이 필요할 경우, VPN tunnel을 이용할 것
3. data를 text 파일로 관리해야 하기 때문에 많은 account를 관리해야 할 경우 권장하지 않음. 꼭 해야 한다면 별도의 관리 프로그램을 만들기를 권장
4. Multi master 구성이 안됨

> ***참고!***  
> ***NIS+*** 는 secure protocol을 지원합니다만, ***NIS***와 ***NIS+***는 별개의 서비스 입니다. 또한, ***NIS+***는 Solaris에서 사용되던 서비스를 Linux에서 사용할 수 있도록 porting이 된 것이 있지만 널리 사용되지 않아 개발이 더디고 문서화가 잘 안되어 있으며, 설정이 복잡하여 리눅스에서는 널리 사용되지 않습니다.


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

보통 NIS 설정을 할때 시스템 상의 */etc/passwd*와 */etc/group*을 이용하여 database를 만드는 것을 권장하는데, 여기서는 이 파일들을 사용하지 않고 */var/yp/etc*에 별도의 파일로 관리하도록 기술 합니다.

###3.3.1 passwd/group list 파일 준비

NIS에서 관리할 passwd/group 파일은 */etc/passwd*와 */etc/group* 과 동일한 format 을 사용합니다.

passwd와 shadow 파일을 관리하기 위해서는 다음의 script를 생성합니다.

```bash
[root@an3 ~]$ mkdir -p /var/yp/etc
[root@an3 ~]$ cat > /var/yp/etc/adduser <<EOFF
#!/bin/bash

# whether use SHADOW system
USE_SHADOW=1

# NIS config directory
ETCDIR=/var/yp/etc

# crypt method (md5|sha256|sha512)
METHOD="md5"

usage() {
    echo "Usage  : \$0 USER_ID [ACCOUNT_NAME]"
    echo "Example: \$0 john \"John smith\""
    exit
}

if [ \$# -lt 1 -o \$# -gt 2 ]; then
    usage
fi

account="\$1"
accname="\$2"

pass=\$(genpasswd -m \${METHOD})
[ \$? -eq 1 ] && exit 1

maxuid=\$(cat \${ETCDIR}/passwd | grep -v "^#" | awk -F':' '{print \$3}' | sort -r | head -n1)
uid=\$[ \${maxuid} + 1 ]
# passwd entry가 한개도 없을 경우 초기화
[ $uid -eq 1 ] && uid=10000

if [ \$USE_SHADOW -eq 1 ]; then
    pent="x"
else
    pent="\${pass}"
fi

cat >> \${ETCDIR}/passwd <<EOF
\${account}:\${pent}:\${uid}:10000:\${accname}:/home/\${account}:/bin/bash
EOF

if [ \$USE_SHADOW -eq 1 ]; then
  chgdate="\$[ \$(date +"%s") / 86400 ]"; echo \$a

  cat >> \${ETCDIR}/shadow <<EOF
\${account}:\${pass}:\${chgdate}:0:99999:7:::
EOF
endif
EOFF
[root@an3 ~]$ chmod 700 /var/yp/etc/adduser
```

***adduser*** script는 기본으로 MD5 방식의 암호를 생성합니다. 만약 연동할 시스템들이 sha512를 지원하는 버전으로만 구성이 되어 있다면 (예를 들어 CentOS/RHEL 5는 sha512를 지원하지 않습니다.), adduser script의 ***METHOD*** 변수 값을 ***sha512***로 수정 하는 것을 권장 합니다.


다음 passwd, group, shadow 파일을 생성 합니다.

```bash
[root@an3 ~]$ cd /var/yp/etc
[root@an3 etc]$ ./adduser USERID "USER NAME"
New Password:
Retype New Password:

[root@an3 etc]$ cat /var/yp/etc/passwd
USERID:x:10000:10000:USER NAME:/home/USERID:/bin/bash
[root@an3 etc]$ cat /var/yp/etc/shadow
USERID:$1$pAhn0Osq$hY4yCJs4mvBOTg6sxmmjM/:16894:0:99999:7:::
[root@an3 etc]$ echo "nisusers:x:10000:" >> /var/yp/etc/group
```

/etc/shadow를 해제하는 것 처럼 /var/yp/etc/shadow는 ***pwunconv***의 *-l* 옵션을 사용할 수 있습니다. 이 옵션은 안녕 리눅스의 pwconv/pwunconv 에만 patch가 되어 있습니다.

```bash
[root@an3 ~]$ pwunconv -l /var/yp/etc
[root@an3 ~]$ ls
passwd    passwd-
[root@an3 ~]$ pwconv -l /var/yp/etc
passwd    passwd-    shadow
```

만약 동작을 하지 않거나 에러 메시지가 나온다면, yum update 명령을 이용하여 ***shadow-tuils*** package를 최신 버전으로 업그레이드 하십시오.



> ***!참고***   
> Web server에서도 ***NIS*** 인증을 사용하기 위하여 apache 또는 lighttpd, nginx의 nis 모듈을 사용할 계획이라면, ***shadow system***을 사용할 수 없습니다. ***yp_match*** call이 non-root 권한에서는 빈 값을 return 하기 때문에 웹서버를 root 권한으로 실행을 해야합니다. 그러므로 shadow system을 사용하지 말고 구성을 하십시오. 또는, 웹서버의 nis 모듈을 사용하지 말고, cronjob으로 ypcat 명령을 이용하여 .htaccess file을 주기적으로 갱신하여 사용하도록 하십시오.
> 
>또한, 보안을 고려 한다면 ***NIS*** 보다는 ***ldap***을 이용하여 구성하는 것을 고려 하십시오.


###3.3.2 /var/yp/Makefile 설정

***/var/yp/Makefile*** 중에서 다음의 설정들을 수정합니다.

```bash
# slave NIS를 구성할 것이라면 값을 true로 변경 합니다. 기본값은 false 입니다.
# slave NIS를 구성할 것이 아니라면 false로 나두십시오.
NOPUSH=true

# 인증 통합시에 system uid/gid와 충돌할 경우가 발생할 수 있습니다. 그러므로
# 충분한 값을 주도록 합니다. 대략 10000번 이상대를 사용하면 거의 충돌할 일이
# 없습니다.
# 이 값은 passwd나 group 파일에 지정한 uid나 gid보다 작으면 database에 포함시키지
# 않음을 의미합니다.
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
# ***shadow*** 추가하고, ***netid***를 제거 하였습니다.
all:  passwd shadow group hosts rpc services protocols mail \
    # netgrp shadow publickey networks ethers bootparams printcap netid \
    # amd.home auto.master auto.home auto.local passwd.adjunct \
    # timezone locale netmasks
```

###3.3.3 /etc/sysconfig/yppasswdd 설정

***/etc/sysconfig/yppasswd*** 를 다음과 같이 수정 합니다.

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


###3.4 보안 설정

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
# YPPASSWDD udp/836
TCP_HOSTPERPORT = 192.168.0.0/24:111 192.168.0.0/24:834-835
UDP_HOSTPERPORT = 192.168.0.0/24:111 192.168.0.0/24:834-836
  ... 하략 ...
```

###3.5 Daemon 실행

```bash
[root@an3 ~]$ service rpcbind start
[root@an3 ~]$ service ypserv start
[root@an3 ~]$ service ypxfrd start
[root@an3 ~]$ service yppasswdd start
[root@an3 ~]$ # booting 시에 시작 되도록 설정
[root@an3 ~]$ systemctl enable rpcbind ypbind ypxfrd yppasswdd
```

###3.6 Database 초기화

daemon을 실행 했으면 database를 초기화 합니다.

```bash
[root@fork yp]# /usr/lib64/yp/ypinit -m

At this point, we have to construct a list of the hosts which will run NIS
servers.  fork.kldp.org is in the list of NIS server hosts.  Please continue to add
the names for the other hosts, one per line.  When you are done with the
list, type a <control D>.
        next host to add:  nis1.domain.com
        next host to add:  <종료를 하기 위하여 CTRL-D 를 누릅니다.>
The current list of NIS servers looks like this:

nisdomain.com

Is this correct?  [y/n: y]  y
We need a few minutes to build the databases...
Building /var/yp/OOPS-NIS/ypservers...
Running /var/yp/Makefile...
gmake[1]: Entering directory `/var/yp/OOPS-NIS'
Updating passwd.byname...
Updating passwd.byuid...
Updating group.byname...
Updating group.bygid...
Updating hosts.byname...
Updating hosts.byaddr...
Updating rpc.byname...
Updating rpc.bynumber...
Updating services.byname...
Updating services.byservicename...
Updating protocols.bynumber...
Updating protocols.byname...
Updating mail.aliases...
gmake[1]: Leaving directory `/var/yp/OOPS-NIS'

fork.kldp.org has been set up as a NIS master server.

Now you can run ypinit -s nis1.domain.com on all slave server.
[root@an3 ~]$
```

##3.7 database update

***ypinit*** 명령은, 최초에 한번만 실행을 해 주면 됩니다. 그 이후, passwd (/var/yp/etc/passwd)나 group (/var/yp/etc/group) 파일을 수정 한 후에는 ***/var/yp*** 에서 *make* 명령을 실행하면 database가 갱신이 됩니다.

```bash
[root@an3 ~]$ cd /var/yp
[root@an3 ~]$ make
gmake[1]: Entering directory '/var/yp/KLDP-NIS'
gmake[1]: 'ypservers'는 이미 갱신되었습니다.
gmake[1]: Leaving directory '/var/yp/KLDP-NIS'
gmake[1]: Entering directory '/var/yp/KLDP-NIS'
Updating passwd.byname...
Updating passwd.byuid...
gmake[1]: Leaving directory '/var/yp/KLDP-NIS'
[root@an3 ~]$
```


##4. NIS slave 설정

master server는 ***nis1.domain.com***, slave server는 ***nis2.domain.com***으로 가정을 합니다.

###4.1 package 설치

```bash
[root@an3 ~]$ yum install ypserv rpcbind
```

###4.2 NIS domain 설정

NIS master에서 설정했던 NIS DOMAIN을 동일하게 설정을 합니다.

```bash
[root@an3 ~]$ ypdomainname OOPS-NIS
[root@an3 ~]$ echo "NISDOMAIN=\"OOPS-NIS\"" >> /etc/sysconfig/network
```

###4.3 보안 설정

NIS 질의를 할 수 있는 네트워크 대역을 제한 합니다. 형식은 ***NETMASK NETWORK*** 형식으로 설정 합니다. 다음의 설정은 127.0.0.0/8 과 192.168.0.0/24 네트워크에서 NIS 질의에 응답하도록 설정한 것입니다.

```bash
[root@an3 ~]$ cat > /var/yp/securenets <<EOF
255.0.0.0 127.0.0.0
255.255.255.0 192.168.0.0
EOF
[root@an3 ~]$
```

방화벽을 사용한다면 ypserv의 포트를 고정 시키고 port를 열어주어야 합니다.

```bash
[root@an3 ~]$ cat >> /etc/sysconfig/network <<EOF
YPSERV_ARGS="-p 834"
EOF
[root@an3 ~]$ 
[root@an3 ~]$ cat /etc/oops-firewall/filter.conf
  ... 상략 ...
# RPCBIND tcp/udp 111
# YPSERV tcp/udp 834
TCP_HOSTPERPORT = 192.168.0.0/24:111 192.168.0.0/24:834
UDP_HOSTPERPORT = 192.168.0.0/24:111 192.168.0.0/24:834
  ... 하략 ...
```

###4.4 Daemon 실행

slave server에서는 ***ypxfrd***와 ***yppasswdd***는 구동하지 않습니다.

```bash
[root@an3 ~]$ service rpcbind start
[root@an3 ~]$ service ypserv start
[root@an3 ~]$ # booting 시에 시작 되도록 설정
[root@an3 ~]$ systemctl enable rpcbind ypserv
```

###4.5 Slave database 초기화

***/usr/lib64/yp/ypinit***를 이용하여 초기화를 합니다.

```bash
[root@an3 ~]$ /usr/lib64/yp/ypinit -s nis1.domain.com
We will need a few minutes to copy the data from nis1.domain.com.
Transferring rpc.bynumber...
Trying ypxfrd ... success

Transferring passwd.byname...
Trying ypxfrd ... success

Transferring protocols.bynumber...
Trying ypxfrd ... success

Transferring mail.aliases...
Trying ypxfrd ... success

Transferring ypservers...
Trying ypxfrd ... success

Transferring shadow.byname...
Trying ypxfrd ... success

Transferring services.byservicename...
Trying ypxfrd ... success

Transferring hosts.byname...
Trying ypxfrd ... success

Transferring hosts.byaddr...
Trying ypxfrd ... success

Transferring group.byname...
Trying ypxfrd ... success

Transferring protocols.byname...
Trying ypxfrd ... success

Transferring services.byname...
Trying ypxfrd ... success

Transferring passwd.byuid...
Trying ypxfrd ... success

Transferring group.bygid...
Trying ypxfrd ... success

Transferring rpc.byname...
Trying ypxfrd ... success


nis2.domain.com''s NIS data base has been set up.
If there were warnings, please figure out what went wrong, and fix it.

At this point, make sure that /etc/passwd and /etc/group have
been edited so that when the NIS is activated, the data bases you
have just created will be used, instead of the /etc ASCII files.
[root@an3 ~]$
```

master와 slave의 architecure가 다를 경우에는 ypxfrd 관련 에러가 발생합니다만, slave 초기화에는 문제는 없으니 참고 하십시오. (예를 들어, master는 x86_64 machine, slave는 x86 machine 일 경우)


###4.6 map 동기화 crontab 설정

NIS databse MAP 동기화를 위하여 다음의 설정을 합니다. 이 cronjob은 master에서 업데이트가 된 시점에서 slave가 다운이 되어서 업데이트가 안된 경우라도 대부분의 NIS map들이 최근 것으로 update 되는 것을 보장 합니다.

```bash
[root@an3 ~]$ cat > /etc/cron.d/yp-slave <<EOF
# YP slave cron 작업 설정
#
# 필드 설명
# Minutes Hour Date Month Week User Command

20 *    * * * root /usr/lib64/yp/ypxfr_1perhour >& /dev/null
40 6    * * * root /usr/lib64/yp/ypxfr_1perday >& /dev/null
55 6,18 * * * root /usr/lib64/yp/ypxfr_2perday >& /dev/null
EOF
```

###4.7 Slave server 등록

master server (nis1.domain.com)에서 slave server를 등록합니다.

```bash
[root@an3 ~]$ hostname
nis1.domain.com
[root@an3 ~]$ echo "nis2.domain.com" >> /var/yp/ypservers
[root@an3 ~]$ cat /var/yp/ypservers
nis1.domain.com
nis2.domain.com
[root@an3 ~]$ service ypserv restart
[root@an3 ~]$
```


##5. NIS client 설정

###5.1 package 설치

```bash
[root@an3 ~]$ yum install ypbind rpcbind
```

###5.2 NIS domain 설정

NIS master에서 설정했던 NIS DOMAIN을 동일하게 설정을 합니다.

```bash
[root@an3 ~]$ ypdomainname OOPS-NIS
[root@an3 ~]$ echo "NISDOMAIN=\"OOPS-NIS\"" >> /etc/sysconfig/network
```

###5.3 NIS 설정

####5.3.1 authconfig 를 이용한 방법

***authconfig***를 이용하여 ***NIS*** 설정을 *enable* 시킵니다.

```bash
[root@an3 ~]$ authconfig --enablenis \
                       --nisdomain=OOPS-NIS \
                       --nisserver=nis1.domain.com \
                       --enablemkhomedir \
                       --update
```

*** NIS slave***를 구성했다면 */etc/yp.conf* 에 추가해 줍니다.

```bash
[root@an3 ~]$ echo "server nis2.domain.com" >> /etc/yp.conf
[root@an3 ~]$ cat /etc/yp.conf
  .. 상략 ..
domain OOPS-NIS server nis1.domain.com
server nis2.domain.com
[root@an3 ~]$
```

다음 ypbind를 재시작 해 줍니다.

```bash
[root@an3 ~]$ service ypbind restart
NIS 서비스 종료 중:                                        [  OK  ]
NIS 서비스 시작 중:                                        [  OK  ]
NIS 서비스를 바인딩하는 중:                                 [  OK  ]
[root@an3 ~]
```

####5.3.2 수동으로 설정

수동 설정은 다음의 

  1. ypbind 설정
  2. login 시에 자동으로 home directory 생성 (PAM)
  3. nsswitch 설정
  4. authconfig 설정

작업들을 진행 합니다.


#####5.3.2.1 ypbind 설정

```bash
[root@an3 ~]$ cat >> /etc/yp.conf <<EOF
domain OOPS-NIS
server nis1.domain.com
server nis2.domain.com
EOF
[root@an3 ~]$
```

####5.3.2.2 PAM 설정

NIS account 계정으로 로그인 시에, home directory가 존재하지 않을 경우 자동으로 생성하도록 합니다.

/etc/pam.d/{system-auth,password-auth,fingerprint-auth,smartcard-auth}-sc 의 session 설정에 아래와 같이 pam_mkhomedir 설정을 해 주십시오.

```bash
[root@an3 ~]$ cat /etc/pam.d/password-auth-ac
  .. 상략..
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_mkhomedir.so umask=0077
session     required      pam_unix.so
[root@an3 ~]$
```

home directory 생성 실패 시 login을 불허 할 것이라면 ***required***로 설정을 하고, home directory 생성 실패를 하더라도 login을 허락할 것이라면 ***optional***로 설정 하십시오. 위의 예제는 4개의 파일 설정이 모두 동일함으로 password-auth-sc 만 예시를 보여 줍니다.

또한, 인증과 암호 변경을 위하여 ***password*** entry에 nis 설정을 추가 합니다. 역시 모든 파일 공통 사항입니다.

```bash
[root@an3 ~]$ cat /etc/pam.d/password-auth-ac
  ... 상략 ...
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= difok=1 minlen=8 minclass=3
password    sufficient    pam_unix.so sha512 shadow nis nullok try_first_pass use_authtok remember=4
password    required      pam_deny.so
  ... 하략 ...
[root@an3 ~]$
```

***shdow*** 뒤에 ***nis***를 추가해 주십시오. pam 설정 변경은 따로 ypbind 재시작이나 서버 rebooting 등의 작업이 필요없이 실시간으로 반영이 됩니다.


####5.3.2.3 nsswitch.conf 설정

passwd, shadow, group 항목에 nis를 추가해 줍니다. nsswitch.conf 에 nis가 등록이 되어야 NIS와 system 인증이 연동이 됩니다.

```bash
[root@an3 ~]$ cat /etc/nsswitch.conf
  .. 상략
passwd:     files nis
shadow:     files nis
group:      files nis
  .. 하략 ..
[root@an3 ~]$
```

####5.3.2.4 authconfig 설정 반영

***/etc/sysconfig/authconfig***의 다음 값들을 변경해 줍니다. 다른 설정을 지우라는 것이 아니라 아래에 보여진 변수의 값을 ***no***에서 ***yes***로 변경 하십시오.

```bash
[root@an3 ~]$ cat /etc/sysconfig/authconfig
USEMKHOMEDIR=yes
USENIS=yes
[root@an3 ~]$
```

####5.3.3 NIS 연동 확인

***ypbind*** daemon을 재시작 합니다.

```bash
[root@an3 ~]$ service ypbind restart
NIS 서비스 종료 중:                                        [  OK  ]
NIS 서비스 시작 중:                                        [  OK  ]
NIS 서비스를 바인딩하는 중:                                 [  OK  ]
[root@an3 ~]
```

NIS password entry와 shadow entry를 받아오는지 확인을 해 봅니다.

```bash
[root@an3 ~]$ ypcat passwd.byname
USERID:x:10000:10000:USER NAME:/home/USERID:/bin/bash
[root@an3 ~]$ ypcat shadow.byname
USERID:$1$pAhn0Osq$hY4yCJs4mvBOTg6sxmmjM/:16894:0:99999:7:::
[root@an3 ~]$ ypcat group.byname
nisusers:x:10000:
[root@an3 etc]$
```

system 에 연동이 잘 되었는지 ***getent*** 명령으로 확인을 해 봅니다. /etc/passwd 내용 아라애 NIS passwd entry가 추가로 나와야 합니다. NIS에서는 10000 이상의 UID를 사용하므로, 10000대의 UID가 있는지 확인해 보시면 됩니다.

UID 확인을 쉽게 하기 위하여 ***pwck*** 명령으로 passwd file을 UID 순서대로 정렬한 후에 ***getent***를 실행 합니다.

```bash
[root@an3 ~]$ pwck -s && grpck -s
[root@an3 ~]$ getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nscd:x:28:28:NSCD Daemon:/:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
avahi-autoipd:x:170:170:Avahi IPv4LL Stack:/var/lib/avahi-autoipd:/sbin/nologin
polkitd:x:997:995:User for polkitd:/:/sbin/nologin
systemd-network:x:998:996:systemd Network Management:/:/sbin/nologin
systemd-bus-proxy:x:999:997:systemd Bus Proxy:/:/sbin/nologin
USERID:x:10000:10000:USER NAME:/home/USERID:/bin/bash
[root@an3 ~]$
```

### 5.3.4 계정 암호 변경

***NIS master*** server에 *yppasswdd* daemon이 동작하고 있다면 NIS client에서 암호 변경이 가능 합니다.

NIS account들의 암호 변경은 일반 account의 암호를 변경하는 것과 같이 ***passwd*** 명령을 이용하여 변경을 하면 됩니다.

root account를 이용하여 다른 NIS account의 암호를 변경하는 경우에는, ***passwd*** 명령으로 변경이 불가능 합니다. 이 때에는 yppasswd 명령을 이용하여 변경을 할 수 있습니다.

```bash
[root@an3 ~]$ passwd joungkyun
joungkyun 사용자의 비밀 번호 변경 중
새  암호:
새  암호 재입력:
NIS 암호는 변경할 수 없습니다.
passwd: 인증 토근 수정 오류
[root@an3 ~]$ yppasswd joungkyun
Changing NIS account information for joungkyun on cnt1.domain.com.
Please enter root password:
Changing NIS password for joungkyun on cnt1.domain.com.
Please enter new password:
Please retype new password:

The NIS password has been changed on cnt1.domain.com.

[root@an3 ~]$
```