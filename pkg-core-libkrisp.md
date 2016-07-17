# libkrisp

### Description:

IPv4 주소 또는 32bit long IP에 해당하는 국가/ISP 정보를 반환하는 라이브러리

### Features:
* **krisplookup**
  ```bash
[root@an3 x86_64]$ krisplookup --help
krisplookup v3.1.4: Resolved korea range ISP inforamtion
Usage: krisplookup [option] ip-address
Options:
         -f path, --datafile=path     set user define database file
         -h , --help                  print this message
         -i , --isp                   only print isp code
         -n , --nation                only print nation code
         -r , --range=[country|isp]   Print all range of Country or ISP
                                      about current IP
```

### Reference:
* https://github.com/Joungkyun/libchardet/blob/master/README.md

### Dependencies:
* [libipcalc](pkg-core-libipcalc.md)
* sqlite

### Sub Packages:
* **libkrisp-devel**  
  libkrisp를 link하기 위한 library와 header 파일들
* **libkrisp-data**  
  libkrisp data 파일

### Releated Packages:
* [httpd-krisp](pkg-core-httpd-krisp.md)
* [perl-KRISP](pkg-core-perl-KRISP.md)
* [php-krisp](pkg-core-php-krisp.md)
* [php-krisp](pkg-core-php-krisp.md)
* [python-krisp](pkg-core-python-krisp.md)
