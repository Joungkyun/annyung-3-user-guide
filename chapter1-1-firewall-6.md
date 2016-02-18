# 6. 특정 국가에서의 접속 제어

안녕 리눅스는 1.0 부터 IP 기반의 Geo data를 기본으로 제공하고 있습니다.

**_oops-firewall_**에서 국가 기반의 filtering을 하기 위해서는 Netfilter용 GeoIP database를 생성해야 합니다.

안녕 리눅스에서는 이 작업을 간편하게 하기 위하여 **_geoip-csv2bin_** 이라는 실행 명령을 제공합니다.

> **주의!!**  
> GeoIP database를 배포하는 Maxmind의 경우, 연속해서 2번 이상의 다운로드를 시도하면 하루 동안 IP를 block 시켜 버립니다. 그렇기 때문에 여러 대에서 GeoIP database를 사용한다면 각 서버에서 다운로드를 받지 말고, 한대의 서버에서 다운로드를 받은 후에, 사용할 서버들로 배포하는 형식을 취하시기 바랍니다. 안 그러면 금방 block 되어 버려 업데이트가 불가능해 집니다.

> Maxmind의 Geo lite(free) database는 한달에 1회 갱신 되며, 보통 4~8일 사이에 업데이트가 됩니다. 그러므로 cronjob에 8~10일 사이에 1회 업데이트 되도록 설정해 주시면 됩니다.

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
    ```
    * GeoIP.dat             - IPv4 Country database
    * GeoIPASNum.dat        - IPv4 Network AS Number database
    * GeoIPASNumv6.dat      - IPv6 Network AS Number database
    * GeoIPv6.dat           - IPv6 Country database
    * GeoLiteCity.dat       - IPv4 City database (한국 환경에서는 매우 부정확함)
    * GeoIPCountryWhois.csv - IPv4 Country database CVS format
    * GeoIPv6.csv           - IPv6 Country database CVS format
    ```
2. GeoIPCountryWhois.csv 와 GeoIPv6.csv를 이용하여 netfilter용 database 생성
    ```
    * /usr/share/GeoIP/BE/  - Big endian을 사용하는 cpu용)
    * /usr/share/GeoIP/LE/  - intel machine용
    ```
3. GeoIPCountryWhois.csv 와 GeoIPv6.csv를 이용하여 netfilter용 이전 database 생성
    ```
    * 안녕 리눅스 1/2 에서 사용하는 format
    * geoipdb.bin
    * geoipdb.idx
    ```

GeoIP database가 준비 되었다면, **_/etc/oops-firewall/user.conf_** 에서 다음의 rule을 이용하여 관리할 수 있습니다.

  ```bash
  # Rusia로 부터의 접근을 모두 막는다.
  %-A INPUT -m geoip --src-cc RU -j DROP
  
  # 한국을 제외하고 ssh 연결을 모두 막는다.
  %-A INPUT -p tcp --dport 22 -m geoip ! --src-cc KR -j DROP
  ```
  
다음, geoip database 업데이트를 Cronjob에 등록해 줍니다.

  ```bash
  [root@an3 ~] cat /etc/cron.d/geoip-update
  # GeoIP database update cronjob
  #
  # GeoIP Lite/Free database는 매월 4~7일 사이에 업데이트가 됩니다.
  # 그러므로, 한국 시간으로 8~10 사이에 업데이트를 등록해 놓으면 됩니다.
  #
  # Example of job definition:
  # .---------------- minute (0 - 59)
  # |  .------------- hour (0 - 23)
  # |  |  .---------- day of month (1 - 31)
  # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
  # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
  # |  |  |  |  |
  # *  *  *  *  * user-name  command to be executed

  # 매월 11일 새벽 3시 14분에 실행
  14 3 11 * * root /usr/bin/geoip-update &> /dev/null
  [root@an3 ~]
  ```