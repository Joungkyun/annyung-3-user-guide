# libipcalc

### Description:

IPv4 관련 연산과 서브넷 연산을 위한 라이브러리

### Features:

1. 시작 IP와 마지막 IP가 속한 가장 작은 subnet 계산 가능


### Reference:
* http://svn.oops.org/wsvn/OOPS.libipcalc/trunk/utils/netcalc.c

### Dependencies:
* None

### Sub Packages:

* **libipcalc-devel**  
  libipcalc를 이용한 개발에 필요한 header 및 library file
* **libipcalc-utils**
  * netcalc
  ```bash
[root@an3 ~]$ netcalc --help
netcalc v1.0.1-dev: get network informations
Usage: netcalc [option] ipaddress/[mask|prefix]
       netcalc [option] StartIP EndIP
Options:
         -h, --help             print this message
         -b, --broadcast        print broadcast
         -n, --network          print network
         -m, --mask             print network mask
         -p, --prefix           print network prefix
         -s, --shell-format     print shell format
         -a, --all              print all
[root@an3 ~]$
```
  * ip2long
  ```bash
[root@an3 ~]$ ip2long --help
ip2long v1.0.1-dev: Convert between IPv4 address and Long IP
Usage: ip2long [option] ip-address|LongIP
Options:
         -h, --help             print this message
         -v, --verbose          print both long ip and ipv4
[root@an3 ~]$
```


### Related Packages:
* None