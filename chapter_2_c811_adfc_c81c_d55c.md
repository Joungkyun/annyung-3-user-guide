# Chapter 2. 접근 제한

이 챕터에서는 안녕 리눅스의 **ACL(Access Control List)**에 대해서 설명을 하고자 합니다.

일단 리눅스에서의 ACL은 여러가지 기법이 있습니다. PAM(Plugged Authentification Module)을 이용하여 account 또는 group 별 login 및 chroot 설정이 가능하며, **GeoIP**를 이용한 IP 기반의 설정을 기본으로 제공 합니다.

또한, 안녕 리눅스에서는 웹 접근 제어에 대해서 KRISP, NIS 등, CentOS 보다 다양한 방법을 제시 합니다.

1. account 접근 제어
 2. login 접근 제어
 3. 경로 접근 제어
2. IP base 접근 제어
 1. Shell login 제어
 2. 웹 접근 제어