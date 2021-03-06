# Shell login Control \(with PAM\)

> Sub 목차 \(하위 페이지\) 1. [login 가능한 account 제한](chapter2-2-pam-control-1.md) 2. [login account chroot](chapter2-2-pam-control-2.md) 3. [Google OTP를 이용한 2factor 인증](chapter2-2-pam-control-3.md)

> 참고  
> 이 문서를 먼저 읽을 후, 하위 페이지를 읽으십시오.

PAM 모듈을 이용하여 account access control을 하는 방법에 대하여 기술을 합니다.

일단, account 접근 관리를 하기 전에, 안녕 리눅스에서 처리하고 있는 account 기본 정책에 대해서 설명을 합니다. 이 정책을 먼저 숙지 하시고, 서브 메뉴를 구독 하시기 바랍니다.

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

위와 같이 _**/etc/login.defs.exception**_ 파일에 등록된 account에 대해서는 암호 만료 체크를 하지 않도록 패치가 되어 있습니다.

이 기능은, ISMS 인증을 할 때 _**/etc/login.defs**_를 검사를 하는데, ISMS 기준을 지키면서 위의 문제를 회피하기 위한 해결책 입니다. \(이 부분은 매우 중요한 부분 입니다.\)

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
  * pam\_tally2 명령을 이용하여 확인

    \[root@an3 ~\]$ pam\_tally2 -u root Login Failures Latest failure From root 31 02/15/16 07:15:12 118.129.166.196 \[root@an3 ~\]$ krisplookup 118.129.166.196 118.129.166.196 \(118.129.166.196\): \(주\)엘지유플러스 \(BORANET\) SUBNET : 255.252.0.0 NETWORK : 118.128.0.0 BROADCAST : 118.131.255.255 DB RANGE : 118.128.0.0 - 118.131.255.255 NATION : Korea, Republic of \(KR\) \[root@an3 ~\]$

  * pam\_tally2 명령을 이용한 account locking 해제 \[root@an3 ~\]$ pam\_tally2 -u root -r Login Failures Latest failure From root 31 02/15/16 07:15:12 118.129.166.196 \[root@an3 ~\]$ pam\_tally2 -u root Login Failures Latest failure From root 0 \[root@an3 ~\]$
* 암호 강도 설정
  * pam\_pwquality 모듈 이용.
    * difok=1    - 이전 암호와의 유사도 검사. 이 설정은 기본 설정보다 약화. 기본 10\(50%\)
    * minlen=8   - 최소 길이 8자
    * minclass=3 - 암호 생성시에, 숫자/대문자/소문자/특수문자 중 3가지를 조합해야 함
  * pam\_unix 모듈 이용
    * remember=4 - /etc/security/opasswd에 기록할 이전 암호 수. 이전 암호 사용 방지

마지막으로, root account로 remote login을 하는 것에 대한 설명입니다.

안녕 리눅스는 버전 2부터 SSH를 이용하여 root account의 non-interactive shell로 remote access 가능합니다. 이는 관리의 편이성 때문에 변경을 한 사항인데, private network가 아닌 서버의 경우에는 remote hole이 될 수도 있습니다.

그래서 안녕 리눅스 3 부터는 private network에 대해서만 root account의 non-interactive shell로 접근이 가능 합니다.

_**/root/.bashrc**_ 하단을 보면 다음의 코드로서 제한을 하고 있습니다. \(만약 이 코드가 없다면 rootfiles 버전이 **8.1-11.an3.0.1** 보다 낮은 것이기 때문에 업데이트를 해 주셔야 합니다. 또한, 버전이 같거나 높더라도 이 코드가 보이지 않는다면, _**.bashrc**_와 _**.bash\_profile**_이 관리자에 의해 수정된 경우 입니다. 이 경우에는 새로운 _**.barrc**_와 _**.bash\_profile**_은 _**.bashrc.rpmnew**_와 _**.bash\_profile.rpmnew**_로 생성되어 있게 됩니다.\)

```bash
[root@an3 ~]$ cat .bashrc
   ... 상략 ...
#
# perimt access private network with SSH connection
#
if [ -n "${SSH_CLIENT}" ]; then
    NETWORK_A_CLASS="${SSH_CLIENT%%.*}."
    # console type
    # non-interactive mode         => serial
    # interactive mode (login shell) => pty
    contype="$(/sbin/consoletype 2> /dev/null)"

    #
    # allow with A class
    #
    if [ -z "${LOGIN_ACCESS}" ]; then
        case "${NETWORK_A_CLASS}" in
            "10.")
                #
                # Allow non-interactive shell from private network range
                #
                # If you want to allow only interactive shell
                # [ "${contype}" = "pty" ] && LOGIN_ACCESS="yes"
                #
                # If you want to allow both interactive and non-interactive shell
                # LOGIN_ACCESS="yes"
                #
                [ "${contype}" != "pty" ] && LOGIN_ACCESS="yes"
                ;;
        esac
    fi

    #
    # allow with B Class
    #
    if [ -z "${LOGIN_ACCESS}" ]; then
        # ssh_client format: IP_ADDRESS:CONNECT_PORT
        ssh_client=$(echo ${SSH_CLIENT} | /bin/awk '{print $1":"$3'})
        NETWORK_B_CLASS=$(echo ${ssh_client} | /bin/sed -e 's/\(\([0-9]\+\.\)\{2\}\).*/\1/g' 2> /dev/null)
        NETWORK_C_CLASS=$(echo ${ssh_client} | /bin/sed -e 's/\(\([0-9]\+\.\)\{3\}\).*/\1/g' 2> /dev/null)

        case "${NETWORK_B_CLASS}" in
            "172.16."|"192.168.")
                #
                # Allow non-interactive shell from private network range
                #
                [ "${contype}" != "pty" ] && LOGIN_ACCESS="yes"
                ;;
        esac
    fi
    #
    # allow with C Class
    #
    if [ -z "${LOGIN_ACCESS}" ]; then
        case "${NETWORK_C_CLASS}" in
            #"211.37.6.")
            #   [ "${contype}" != "pty" ] && LOGIN_ACCESS="yes"
            #   ;;
            *)
                #
                # allow per host
                #
                case "${ssh_client}" in
                    #"1.1.1.1:22")
                    #   [ "${contype}" != "pty" ] && LOGIN_ACCESS="yes"
                    #   ;;
                    #"143.1.1.1:2020")
                    #   [ "${contype}" != "pty" ] && LOGIN_ACCESS="yes"
                    #   ;;
                    *)
                        LOGIN_ACCESS=""
                esac
                ;;
        esac
    fi

    [ -z "$LOGIN_ACCESS" ] && \
        echo -en "* \\033[1;31mNotice:\\033[0;39m" && \
        echo " You can't access root privileges with remote access!" && \
        exit
fi
[root@an3 ~]$
```

shell script를 할 줄 아시는 분들은 쉽게 이해가 되겠지만, 간단히 설명을 하자면, private ip 대역 \(10.0.0.0/8, 172.16.0.0/16, 192.168.0.0/16\) 대역에 대해서는 기본으로 non-interactive shell 접근을 가능하게 하고 나머지는 막는 스크립트 입니다. 만약 열어줘야 할 public network이 있다면 설정을 추가해 줄 수 있습니다만, 기본으로 사용하는 것을 권장 합니다.

안녕 리눅스의 shell login에 대하여 이해를 하였다면, 다음 기술 문서를 이용하여 shell login에 대해서 더 정교하게 access 설정을 할 수 있습니다.

1. [login 가능한 account 제한](chapter2-2-pam-control-1.md)
2. [login account chroot](chapter2-2-pam-control-2.md)
3. [Google OTP를 이용한 2factor 인증](chapter2-2-pam-control-3.md)

