# GeoIP

### Descriptions:
Geoip 를 이용한 국가코드를 알아내는 오픈소스 C API

### Changes on AnNyung:
1. 1.6.9 update
2. xt_geoip_build 추가
3. geoip-csv2bin
 * geoip-csvupdate를 실행한 후, cvs2bin 및 xt_geoip_build를 실행하여 kmod-geoip용 database를 생성한다.
4. geoip-csvupdate
 * GeoIP-lite, GeoIPv6, GeoLiteCity, GeoIPASNum, GeoIPASNumv6 database를 업데이트 한다.

### Sub packages:
* **GeoIP-devel** - GeoIP 를 이용하기 위한 개발툴
* **GeoIP-doc** - GeoIP 문서
* **GeoIP-update** - GeoIP database update tool

### Releated Packages:
* [**kmod-geoip**](pkg-core-kmod-geoip)