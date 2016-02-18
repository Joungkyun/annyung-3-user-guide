# 6. 특정 국가에서의 접속 제어

안녕 리눅스는 1.0 부터 IP 기반의 Geo data를 기본으로 제공하고 있습니다.

**_oops-firewall_**에서 국가 기반의 filtering을 하기 위해서는 Netfilter용 GeoIP database를 생성해야 합니다.

안녕 리눅스에서는 이 작업을 간편하게 하기 위하여 **_geoip-csv2bin_** 이라는 실행 명령을 제공합니다.

```bash
[root@an3 ~]$ cd /usr/share/GeoIP
[root@an3 GeoIP]$ /usr/bin/geoip-csv2bin
Get GeoLiteCountry/GeoIP.dat.gz .. OK
Uncompressd GeoIP.dat.gz .. OK
Copying GeoIP.dat .. OK
Get GeoLiteCity.dat.gz .. OK
Uncompressd GeoLiteCity.dat.gz .. OK
Copying GeoLiteCity.dat .. OK
Get GeoIPCountryCSV.zip .. OK
Uncompressd GeoIPCountryCSV.zip .. OK
Copying GeoIPCountryWhois.csv .. OK
Get GeoIPv6.csv.gz .. OK
Uncompressd GeoIPv6.csv.gz .. OK
Copying GeoIPv6.csv .. OK
Get GeoIPv6.dat.gz .. OK
Uncompressd GeoIPv6.dat.gz .. OK
Copying GeoIPv6.dat .. OK
Get asnum/GeoIPASNum.dat.gz .. OK
Uncompressd GeoIPASNum.dat.gz .. OK
Copying GeoIPASNum.dat .. OK
Get asnum/GeoIPASNumv6.dat.gz .. OK
Uncompressd GeoIPASNumv6.dat.gz .. OK
Copying GeoIPASNumv6.dat .. OK
139987 entries total
    0 IPv6 ranges for A1 Anonymous Proxy
   91 IPv4 ranges for A1 Anonymous Proxy
    0 IPv6 ranges for A2 Satellite Provider
  329 IPv4 ranges for A2 Satellite Provider
    9 IPv6 ranges for MW Malawi
  ** 중략 **
  136 IPv6 ranges for ZA South Africa
  667 IPv4 ranges for ZA South Africa
   11 IPv6 ranges for ZM Zambia
   64 IPv4 ranges for ZM Zambia
   10 IPv6 ranges for ZW Zimbabwe
   58 IPv4 ranges for ZW Zimbabwe
[root@an3 GeoIP]$ l
합계 47300
drwxr-xr-x 2 root root    12288  1월 23 17:23 BE/
-rw-r--r-- 1 root root   825130  1월 29 19:37 GeoIP-initial.dat
-rw-r--r-- 1 root root   835223  2월 18 19:53 GeoIP.dat
-rw-r--r-- 1 root root  4079692  2월 18 19:53 GeoIPASNum.dat
-rw-r--r-- 1 root root  4914518  2월 18 19:53 GeoIPASNumv6.dat
-rw-r--r-- 1 root root  8505106  2월  2 22:40 GeoIPCountryWhois.csv
-rw-r--r-- 1 root root  4182797  2월 18 19:53 GeoIPv6.csv
-rw-r--r-- 1 root root  1567099  2월 18 19:53 GeoIPv6.dat
-rw-r--r-- 1 root root 19136630  2월 18 19:53 GeoLiteCity.dat
drwxr-xr-x 2 root root    12288  1월 29 18:06 LE/
-rwxr-xr-x 1 root root    13897  1월 29 19:37 csv2bin
-rw-r--r-- 1 root root  1046148  2월 18 19:53 geoipdb.bin
-rw-r--r-- 1 root root     1470  2월 18 19:53 geoipdb.idx
[root@an3 GeoIP]$
```

**_geoip-csv2bin_** 명령을 실행을 하면 다음의 작업을 하게 됩니다.

1. GeoIP database 갱신
  * GeoIP.dat
  * GeoIPASNum.dat
  * GeoIPASNumv6.dat
  * GeoIPv6.dat
  * GeoLiteCity.dat
  * GeoIPCountryWhois.csv
  * GeoIPv6.csv
2. GeoIPCountryWhois.csv 와 GeoIPv6.csv를 이용하여 netfilter용 database 생성
  * /usr/share/GeoIP/BE/* (Big endian을 사용하는 cpu용)
  * /usr/share/GeoIP/LE/* (intel machine용)
3. GeoIPCountryWhois.csv 와 GeoIPv6.csv를 이용하여 netfilter용 이전 database 생성
  * 안녕 리눅스 1/2 에서 사용하는 format
  * geoipdb.bin
  * geoipdb.idx

