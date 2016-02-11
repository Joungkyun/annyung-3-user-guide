# rootfiles

### Description:
root 홈디렉토리에 필요한 기본 파일

### Changes on AnNyung:
1. root account를 tty console로 login 하지 못하도록 .bash_profile 에서 막음

  ```bash
[root@an3 ~]$ cat /root/.bash_profile
...
[ -z "$SU_USER" ] && [ "`/sbin/consoletype 2> /dev/null`" = "pty" ] && \
        echo -en "* \\033[1;31mNotice:\\033[0;39m" && \
        echo " You can't access root privileges with remote access!" && \
        exit
...
[root@an3 ~]$ 
```

### Sub packages:
