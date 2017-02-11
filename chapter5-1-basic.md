# Chapter 5.1 기본 설정




## 5.1.1 Bind recursion 설정

안녕 리눅스의 bind는 기본으로, ***localhost***에서만 query가 가능 합니다. 그러므로 외부에서 query를 할 수 있도록 하려면 설정을 변경 해야 합니다.

외부에서의 query에 대해서는 ***recursion*** 대한 이해가 필요 합니다.

보통 DNS는 자신에게 설정 되어 있는 도메인 외의 질의 요청이 오면, 해당 도메인이 설정 되어 있는 name server에게 요청을 보낸 후, 응답 받은 내용으로 응답을 합니다. 즉, 기본으로 caching name server 역할을 한다는 의미입니다. 더 쉽게 설명을 하자면, name server에 등록이 되어 있지 않은 도메인에 대한 질의가 들어오더라도 찾아서 응답을 해 준다는 의미입니다.

아래와 같이 KLDP.org의 name server에 google.com의 A record를 질의 하는 경우 입니다.

```bash
[root@an3 named]$ dig google.com @ns.kldp.org

; <<>> DiG 9.9.4-geoip-1.4-RedHat-9.9.4-38.an3.1 <<>> google.com @ns.kldp.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53082
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 5

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       172.217.24.142

;; AUTHORITY SECTION:
google.com.             80178   IN      NS      ns2.google.com.
google.com.             80178   IN      NS      ns1.google.com.
google.com.             80178   IN      NS      ns4.google.com.
google.com.             80178   IN      NS      ns3.google.com.

;; ADDITIONAL SECTION:
ns2.google.com.         80178   IN      A       216.239.34.10
ns1.google.com.         80178   IN      A       216.239.32.10
ns3.google.com.         80178   IN      A       216.239.36.10
ns4.google.com.         80178   IN      A       216.239.38.10

;; Query time: 144 msec
;; SERVER: 14.0.82.80#53(14.0.82.80)
;; WHEN: 토  2월 11 03:30:45 KST 2017
;; MSG SIZE  rcvd: 191

[root@an3 named]$
```

***recursion***은 이 caching name server 역할을 제한 하는 기능을 말합니다. ***recursion*** 옵션이 ***no***로 설정이 되어 있다면 설정이 되어 있는 도메인 외의 요청에 대해서는 ***권한이 없다***는 응답을 하게 되고, 오로지 설정이 되어 있는 도메인에 대해서만 응답을 하게 됩니다.

```bash
oops@linux:~$ dig google.com @ns.kldp.org

; <<>> DiG 9.10.3-P4-Ubuntu <<>> google.com @ns.kldp.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 38503
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google.com.                    IN      A

;; Query time: 7 msec
;; SERVER: 14.0.82.80#53(14.0.82.80)
;; WHEN: Sat Feb 11 03:29:33 KST 2017
;; MSG SIZE  rcvd: 39

oops@linux:~$
```

***recursion*** 기능에 대해서 설명을 하는 이유는, 이 recursion 기능이 public하게 설정이 되어 있을 경우, ***DNS cache poisoning*** 공격을 발생시킬 수 있기 때문입니다. 그러므로 근래의 DNS 설정시에는 이 ***recursion*** 기능을 off 시키는 것을 권장 합니다. 하지만 무조건 ***recursion***을 off 시킬 수 없는 환경이 있을 수 있기 때문에, 이 ***recursion*** 설정 정책에 대해서 설명을 하고자 합니다.

일단 ***recursion*** 설정 정책은 다음의 case 를 생각해 볼 수 있습니다.<br><br>

1. **내 도메인 설정을 위한 DNS와 내부에서 사용할 caching name server가 따로 존재할 경우**<br><br>
   외부에서의 query를 허가하고, ***recursion***을 off 시킵니다. ***/var/named/etc/named.conf***의 ***options*** block에서 다음의 설정을 변경 또는 추가 하십시오. 별도로 있는 caching name server는 3번째 case를 참조 하십시오.
   ```bind
   options {
       ...
       recursion no;
       allow-query { any; };
       ...
   };
   ```
   
2. **caching name server가 따로 존재하지 않을 경우**<br>
   외부에서의 query를 허가하고, ***recursion***을 할 수 있는 대역을 ***allow-recursion*** 옵션을 이용하여 설정 합니다.  
   
   먼저 ***allow-recursion***에 사용할 ***RecursionAllow*** ACL group을 ***/var/named/etc/named.acl.conf***에 생성 합니다.  
   ```bind
   acl LocalAllow {
       127.0.0.1;
       localhost;
   };

   acl RecursionAllow {
       192.168.0.0/16;
       111.112.113.128/25;
   };
   ```
   
   다음, ***named.conf***의 ***options*** block에서 다음의 설정을 변경 또는 추가 합니다.
   ```bind
   options {
       ...
       recursion yes;
       allow-query { any; };
       allow-recursion { LocalAllow; RecursionAllow; };
       ...
   };
   ```
   
3. **내부 전용 caching name server로 사용할 경우**<br>
   내부에서의 query만 허가하고, ***recursion***을 public 하게 설정 합니다.  
   
   먼저 ***/var/named/etc/named.acl.conf***의 ***LocalAllow*** acl group에 query를 할 내부 대역을 추가해 줍니다.
   ```bind
   acl LocalAllow {
       127.0.0.1;
       localhost;
       192.168.0.0/16;
   };
   ```
   다음, ***named.conf***의 ***options*** block에서 다음의 설정을 변경 또는 추가 합니다.
   ```bind
   options {
       ...
       recursion yes;
       allow-query { LocalAllow; };
       ...
   };
   ```
4. **public caching name server로 사용할 경우**<br>
   외부에서의 ***query***와 ***recursion***을 모두 허가합니다. 이 경우에는 DNS 공격을 위하여 ***RPZ***(Response Policy Zone)과 같이 공격을 방어하기 위한 기법들이 필요 합니다. 여기서는 따로 public caching name server 구성에 대해서는 따로 다루지는 않겠습니다.  
   ***named.conf***의 ***options*** block에서 다음의 설정을 변경 또는 추가 합니다.
   ```bind
   options {
       ...
       recursion yes;
       allow-query { any; };
       ...
   };
   ```

***recursion*** 정책 설정이 완료가 되면, 이제 bind는 기본적으로 정책에 따라 응답을 할 수 있는 상황이 됩니다.


## 5.1.2 bind log 설정

안녕 리눅스 3의 bind 기본 log 설정은 다음과 같습니다. bind log 설정은 ***/var/named/etc/named.conf***에 있습니다.

```bind
logging {
    channel default_debug {
        file "/log/named.log";
        severity dynamic;
        print-time yes;
    };

    channel "query-log" {
        file "/log/query.log";
        severity info;
        print-category yes;
        print-time yes;
    };

    category queries { query-log; };
    category lame-servers { null; };
    category unmatched { null; };
    category network { null; };
    category notify { null; };
    category update { null; };
};

```

***/var/log/named/named.log***에 bind daemon 관련 log가 기록되며, 외부 DNS나 client들의 도메인 질의에 대한 log는 ***/var/log/named/query.log***에 기록이 됩니다.

bind logging은 ***channel***을 정의를 하고, 각종 ***category***를 어느 ***channel***로 기록하게 할지를 결정 하면 됩니다. ***channel***은 사용자 정의이기 때문에 알아서 설정을 하면 됩니다.

***channel*** 설정과 ***category*** 항목에 대해서는 다음 문서를 참고 하십시오.

https://ftp.isc.org/isc/bind9/cur/9.9/doc/arm/Bv9ARM.ch06.html#logging_grammar
https://ftp.isc.org/isc/bind9/cur/9.9/doc/arm/Bv9ARM.ch06.html#logging_statement








## 5.1.3 bind 구동 확인

안녕 리눅스 3의 데몬 구동은 systemd를 이용 합니다. 다음의 명령으로 bind 구동이 제대로 되는지 확인을 합니다.

```bash
[root@an3 ~]$ service named restart
```

또는

```bash
[root@an3 ~]$ systemctl restart named.service
```

구동이 잘 되었는지 확인을 하려면, ***/var/log/named/named.og*** 파일을 확인해 보도록 합니다. 실시간으로 확인을 하고 싶으시다면, 구동 전에, ***tail*** 명령을실행한 다음 n ***-f*** 옵션을 주고 ***/var/log/named/named.log***를 실행한 다음 ***bind***를 재시작 해 주시면 됩니다.
