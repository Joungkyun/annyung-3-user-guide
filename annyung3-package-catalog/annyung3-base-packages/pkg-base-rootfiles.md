# rootfiles

## Description:

root 홈디렉토리에 필요한 기본 파일

## Changes on AnNyung:

1. serial console로 접속시 LANG 환경 변수를 en\_US.UTF-8로 변경
2. [ISMS](http://isms.kisa.or.kr/kor/intro/intro01.jsp) 인증 관련 정책 적용
3. root login 정책 제한
   1. 기본적으로 root로 login을 제한
   2. 사설 IP에서의 접근의 경우, tty console 만\(non-intercative shell\) 접속 가능
      * `ssh root@host` 접속 불가
      * `ssh root@host "ls -al"` 가능
   3. /root/.bashrc 에서 사용자 정의 가능 

```bash
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
```

## Sub packages:

