# bind

### Descriptions:
Chroot가 적용된 버클리 인터넷 네임 서버 (BIND)

### Changes on AnNyung:
1. bind-chroot 패키지 제거하고, 기본으로 chroot로 동작하도록 구성 (_/var/named_)
2. _/etc/named/naemd.conf_는 _/var/named/etc/named.conf_의 solft link (호환성 유지)
3. zone file과 _named.conf_에서 IDN을 직접 사용 가능
 * http://annyung.oops.org/?m=white&p=mdns
4. multiple CNAME 지원
5. geodns 기능 지원
 * _/etc/sysconfig/named_에 GEOIP_DATA_COPY="yes" 설정
 * https://code.google.com/p/bind-geoip/wiki/UsageGuide 참조

### Dependencies:
 * [GeoIP](pkg-base-GeoIP.md)

### Sub packages:
 * **bind-devel**- BIND DNS 개발을 위해 필요한 헤더 파일과 라이브러리
 * **bind-libs** - BIND DNS에 필요한 라이브러리
 * **bind-libs-lite** - DNS 프로토콜 동작을 위한 라이브러리
 * **bind-license** - BIND DNS 라이센스
 * **bind-lite-devel** - BIND DNS 개발을 위해 필요한 최소 헤더파일 및 라이브러리
 * **bind-pkcs11** - 암호화를 위한 BIND 내장 PKCS#11 기능
 * **bind-pkcs11-devel** - Development files for Bind libraries compiled with native PKCS#11
 * **bind-pkcs11-libs** - Bind libraries compiled with native PKCS#11
 * **bind-pkcs11-utils** - DNSSEC를 사용하기 위한 BIND 내장 PKCS#11 도구
 * **bind-sdb** - 데이터베이스 백엔드와 DLZ를 지원하는 BIND 서버
 * **bind-utils** - DNS 서버에 질의를 하기 위한 도구