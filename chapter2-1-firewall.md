# Chapter 2. Access Control
## 1. 안녕 리눅스 방화벽 설정

> 목차  
1. [기본 설정](chapter1-1-firewall-1)  
2. [Inbound 제어](chapter1-1-firewall-2)  
3. [Outbound 제어](chapter1-1-firewall-3)  
4. [brute force attack 제어](chapter1-1-firewall-4)  
5. [User defined rule 제어](chapter1-1-firewall-5)
6. [특정 국가에서의 접속 제어](chapter1-1-firewall-6)
7. [oops-firewall 실행 방법](chapter1-1-firewall-7)


---
<br>


안녕 리눅스는 CentOS/RHEL 7이 기본으로 제공하는 [firewalld](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)를 사용하지 않고, 안녕 1.x 부터 제공해 오던 **[oops-firewall](core-pkg-oops-firewall.md)**을 제공합니다.

**oops-firewall**이나 **firewalld**는 모두 iptables를 backend로 하는 즉, iptables rule을 대신 작성해 주는 utility라고 볼 수 있습니다. 즉, **oops-firewall**이나 **firewalld**에서 설정은 iptables rule을 작성한 것이고, 이 rule을 ipatbles로 deploy하여 netfilter에 반영을 하게 되는 것입니다.

한마디로, 어려운 iptables rule을 만들기 쉽게 도와주고, 정책을 생성/제거/반영을 쉽게 도와주는 도구라는 것이기 때문에 익숙한 것을 사용하시면 됩니다.

만약 CentOS/RHEL 7에서 제공하는 **firewalld**를 사용하기 원한다면, 다음의 절차를 따라 주십시오.

```sh
[root@an3 ~]$ yum remove oops-firewall
[root@an3 ~]$ yum install firewalld
[root@an3 ~]$ service firewalld enable
```

상기 작업을 완료 하였다면, **oops-firewall** 대신에 **firewalld**를 사용할 준비가 완료된 상태 입니다. 여기서 부터는 [RHEL 7 System Admin Guidel의 firewalld 설명]((https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)을 참조 하십시오. 이 이하는 **oops-firewall**에 대한 기술을 진행 합니다.

이 문서에서는 **oops-firewall**에 대하여 바로 사용을 할 수 있는 대략적인 기법에 대해서만 기술을 합니다. **oops-firewall**의 전체적인 특징과 자세한 사용 방법은 [**oops-firewall** 사용 설명서](http://oops.org/?t=lecture&sb=firewall&n=2)를 참조 하십시오.

안녕 리눅스에서 **firewalld** 대신 **oops-firewall**을 제공하는 이유는 iptables rule에 대해서 전혀 지식이 없어서 사용을 할 수 있는 간단함과 명료한 설정이 하나이고, iptables를 잘 다를 수 있는 경우, **oops-firewall** 이 만든 rule의 쉽게 customizing을 할 수 있다는 점 입니다.

안녕 리눅스의 구동은 다음과 같습니다.

기본적으로는 RHEL 시스템의 init service 구동 명령인 /sbin/service를 이용하면 됩니다.


    ** 참고!

    다음 용어(표현)를 기억 하십시오.

    Inbound  - 외부(remote) 에서 내부(local)로 들어오는 접근
    Outbound - 내부(local) 에서 외부(remote)로 나가는 접근
    TCP/22   - TCP 22번 포트
    TCP/1.1.1.1:22 - TCP 1.1.1.1 IP의 22번 포트
    
    Rule 표현식:
    ------------
    
    * IP address 표현식
    
     - Anywhere(모든 IP) 표현식
       0.0.0.0/0 = 0/0 = ANYWHERE = anywhere
       
     - IP 범위 표현식
       1.1.1.1-255       => 1.1.1.1-1.1.1.255
       1.1.1.1-2.255     => 1.1.1.1-1.1.2.255
       1.1.1.1-2.3.255   => 1.1.1.1-1.2.3.255
       1.1.1.1-1.2.3.255 => 1.1.1.1-1.2.3.255
       
     - subnet
       10.10.10.0/24
       10.10.10.0/255.255.255.0

    * Port 표현식
      port[:state] => state가 지정되지 않으면 기본으로 NEW를 사용

      53
      53-100 (53번 부터 100번 포트까지의 범위 지정)
      53:NEW
      20:ESTABLISHED,RELATED

      => STATE
         NEW          : 새로운 연결
         ESTABLISHED  : 연결이 성립되어 있는 상태
         RELATED      : FTP의 20번 포트나 passive port와 같은 연관 연결
         INVALID      : 이상 패킷

    * 설정 표현식

      값의 구분자는 공백문자를 사용함.

        설정옵션 = 값1 값2 값3

      다음과 같이 값을 여러줄로 설정이 가능.
      마지막은 '\' 문자가 없어야 함. '\' 문자는 한줄로 연결한다는 의미를 뜻합니다.

        Directive = VALUE1 \
                    VALUE2 \
                    VALUE3
