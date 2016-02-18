# Chapter 2. Access Control
## 2. Shell login Control (with PAM)

PAM 모듈을 이용하여 account access control을 하는 방법에 대하여 기술을 합니다.

일단, account 접근 관리를 하기 전에, 안녕 리눅스에서 처리하고 있는 account 기본 정책에 대해서 설명을 합니다.


안녕 리눅스의 account 기본 정책은 **"개인 정보 보호법"**에 의하여 년 1회의 보안 감사를 ISMS 인증으로 대체를 하게 되면서, ISMS에서 요구하는 사항을 기본으로 반영해 놓고 있습니다.

```bash
[root@an3 ~]$ cat /etc/login.defs
  ** 상략 **
  
#   PASS_MAX_DAYS   Maximum number of days a password may be used.
#   PASS_MIN_DAYS   Minimum number of days allowed between password changes.
#   PASS_MIN_LEN    Minimum acceptable password length.
#   PASS_WARN_AGE   Number of days warning given before a password expires.
#
PASS_MAX_DAYS   90
PASS_MIN_DAYS   0
PASS_MIN_LEN    8
PASS_WARN_AGE   7

  ** 하략 **
[root@an3 ~]$
```

위의 설정은 다음을 의미합니다.

    * 암호 만료 유지 기간 : 90일
    * 암호 만료 경고      : 7일 전부터
    * 암호 최소 길이      : 8자

하지만, LDAP이나 NIS, AD등의 인증 통합을 한 상황에서는 이 설정은 보안상 도움이 되지를 못하고, 다른 문제를 야기시키는 경우가 있습니다.

  1. LDAP/NIS/AD등의 통합인증 시스템에 있는 account가 local account도 있을 경우, loca account password expire에 의해서 locking 되는 문제
  2. root나 작업을 위한 공용 계정들의 경우 sudo나 SSH key를 이용하기 때문에 암호를 거의 사용하지 않음에도 불구하고 제한이 걸려, 암호가 만기 되었을 경우 해당 계정의 cronjob이 중지되는 문제

이 외에도 여러가지 문제가 부수적으로 발생을 합니다. 안녕 리눅스에서는 이런 문제를 해결하기 위하여 PAM과 shadow-utils에 password expire에 예외를 둘 수 있도록 패치가 되어 있습니다.

```bash
[root@an3 ~]$ cat /etc/login.defs.exception
root
work
[root@an3 ~]$
```

위와 같이 ***/etc/login.defs.exception*** 파일에 등록된 account에 대해서는 암호 만료 체크를 하지 않도록 패치가 되어 있습니다.

이 기능은, ISMS 인증을 할 때 ***/etc/login.defs***를 검사를 하는데, ISMS 기준을 지키면서 위의 문제를  회피하기 위한 해결책 입니다. (이 부분은 매우 중요한 부분 입니다.)




다음은, authentification 실패시의 정책과 암호 관리에 대한 정책 입니다.

```bash
[root@an3 ~]$ cat /etc/pam.d/password-auth-sc
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_tally2.so deny=4 unlock_time=120

  ** 중략 **

account     required      pam_tally2.so

  ** 중략 **

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= difok=1 minlen=8 minclass=3
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok remember=4

  ** 하략 **
[root@an3 ~]$
```

위의 설정은 다음을 의미 합니다.

    * 암호가 연속해서 3번 실패했을 경우 account를 120초 동안 잠금
      - pam_tally2 명령을 이용하여 확인
      
        [root@an3 ~]$ pam_tally2 -u root
        Login           Failures Latest failure     From
        root               31    02/15/16 07:15:12  118.129.166.196
        [root@an3 ~]$ krisplookup 118.129.166.196
        118.129.166.196 (118.129.166.196): (주)엘지유플러스 (BORANET)
        SUBNET    : 255.252.0.0
        NETWORK   : 118.128.0.0
        BROADCAST : 118.131.255.255
        DB RANGE  : 118.128.0.0 - 118.131.255.255
        NATION    : Korea, Republic of (KR)
        [root@an3 ~]$
        
      - pam_tally2 명령을 이용한 account locking 해제
        [root@an3 ~]$ pam_tally2 -u root -r
        Login           Failures Latest failure     From
        root               31    02/15/16 07:15:12  118.129.166.196
        [root@an3 ~]$ pam_tally2 -u root
        Login           Failures Latest failure     From
        root                0
        [root@an3 ~]$
        
    * 암호 강도 설정
      - pam_pwquality 모듈 이용.
        + difok=1     - 이전 암호 기억 회수. 이전 암호를 사용하지 못하기 위하여 저장
        + minlen=8    - 최소 길이 8자
        + minclass=3  - 암호 생성시에, 숫자/대문자/소문자/특수문자 중 3가지를 조합해야 함

      

      
      