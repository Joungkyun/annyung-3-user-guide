# Chapter 2. Access Control

이 챕터에서는 안녕 리눅스의 **ACL(Access Control List)**에 대해서 기술 합니다.

**ACL(Access Control List)**이라는 단어는 보통은 Network 관련하여 접근제어를 할 때 많이 사용하는 단어 입니다. 대부분 IP 기반에서 많이 사용하지만 IP base 외에도 account base의 ACL도 가능 합니다.

여기서는 안녕 리눅스를 이용하여 다양한 **ACL(Access Control List)** 설정 기법에 대해서 기술 합니다.

안녕 리눅스의 **ACL(Access Control List)**의 특징 중의 하나는 국가/ISP base의 control에 대해서 적극적으로 지원을 한다는 점입니다.

다룰 대상에 대해서는 다음과 같습니다.

1. [안녕 리눅스 방화벽 설정](chapter2-1-firewall.md)
2. [Shell login Control (with PAM)](chapter2-2-pam-control.md)
  1. [login 가능한 account 제한](chapter2-2-pam-control-1.md)
  2. [login account chroot](chapter2-2-pam-control-2.md)
  3. [Google OPT를 이용한 2 factor 인증](chapter2-2-pam-control-3.md)
3. [인증 통합 (Authentification/Authorization Intergrate)](chapter2-2-pam-control.md)
  1. Openldap
  2. NIS
  3. Active Directory

