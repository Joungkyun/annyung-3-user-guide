# Chapter 2. Access Control

이 챕터에서는 안녕 리눅스의 **ACL(Access Control List)**에 대해서 기술 합니다.

**ACL(Access Control List)**이라는 단어는 보통은 Network 관련하여 접근제어를 할 때 많이 사용하는 단어 입니다. 대부분 IP 기반에서 많이 사용하지만 IP base 외에도 account base의 ACL도 가능 합니다.

여기서는 안녕 리눅스를 이용하여 다양한 **ACL(Access Control List)** 설정 기법에 대해서 기술 합니다.

안녕 리눅스의 **ACL(Access Control List)**의 특징 중의 하나는 국가/ISP base의 control에 대해서 적극적으로 지원을 한다는 점입니다.

다룰 대상에 대해서는 다음과 같습니다.

1. 안녕 리눅스 방화벽 설정
2. Authentification
2. Geo data를 이용한 Network Access control
3. PAM을 이용한 account 기반의 Access control
  1. login account 제한
  2. login account chroot
  3. Google OPT를 이용한 2 factor 인증
4. Web server level의 IP및 account, Country/ISP data 기반의 Access control
  1. Apache
  2. nginx
  3. lighttpd
