# login account chroot

_**pam\_chroot PAM**_ 모듈을 이용하여 _**Jail**_ filesystem 구성하는 방법에 대하여 기술합니다.

RHEL 7, CentOS 7, 안녕 리눅스 3은 원격 접속시에 _**password-auth-sc**_를 이용하며, local에서 su 명령을 이용한 계정 전환시에는 _**system-auth-sc**_를 이용합니다. 그러므로, _**chroot**_를 적용하기 위해서는 이 두 파일을 모두 제어해 주어야 합니다.

원격 접속을 위하여 _**password-auth-sc**_의 session part 최상단에 _**required**_로 _**pam\_chroot**_를 설정 합니다.

```bash
[root@an3 ~]$ cat /etc/pam.d/password-auth-sc
  ** 상략 **

session     required      pam_chroot.so debug
session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
[root@an3 ~]$
```

다음, _**LOCAL**_에서의 계정 전환\(_**su**_\)를 위하여 _**system-auth-sc**_의 session part 최상단에 _**required**_로 _**pam\_chroot**_를 설정 합니다.

```bash
[root@an3 ~]$ cat /etc/pam.d/system-auth-sc
 ** 상략 **

session     required      pam_chroot.so debug
session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
[root@an3 ~]$
```

다음, jail 시킬 account를 _**/etc/security/chroot.conf**_에서 설정 합니다.

```bash
[root@an3 ~]$ cat /etc/security/chroot.conf
# /etc/security/chroot.conf
# format:
#
# Expression           CHROOTDIR
#
# Expression has user or group. You can use regular expression on User, and
# quoted with '/' (For example /^username$/). Group expression is use '%s'
# character at first (For example %groupname)
#
# If first byte is '!', it means chroot not user|group.
# if first byte is '-', it means that don't chroot this user|group.
#
# Configuration is Applied sequential
#
# Users who start sma_ is chrooted with /home/sam
# /^sam_/       /home/sam
#
# sam account is chrooted with /home/sam
# sam           /home/sam
#
# admin account is not chrooted.
# -admin        /home/nodir
#
# If user is not admin group, chroot /home/chroot
# !%admin       /home/chroot
#
# nobody group is chrooted with /home/nobody
# %nobody       /home/nobody

jk  /chroot
[root@an3 ~]$
```

안녕 리눅스의 _**pam\_chroot**_ 모듈은 RHEL/CentOS의 _**pam\_chroot**_ 모듈보다 더 많은 기능을 제공합니다.

상단 _**chroot.conf**_ 의 주석에서 볼 수 있듯이 기존 RHEL의 모듈은

```bash
ACCOUNT   CHROOT_DIR
```

의 형식 밖에 지원하지 않으나, 안녕 리눅스에서는 _**ACCOUNT**_ 부분에 정규식, 그룹, 예외, reverse 지정이 가능 합니다.

```bash
# sam_me 계정은 Jail 하지 않음
-sam_me   /chroot

# sam_ 로 시작하는 모든 계정을 /chroot에 Jail
/^sam_/   /chroot

# sam이 아닌 모든 계정을 /chroot에 Jail
!sam      /chroot

# sam group을 /chroot에 Jail
%sam      /chroot
```

일단, 여기까지 설정을 하면 _**chroot**_가 동작을 합니다.

하지만, _**chroot**_를 하겠다는 것은 모든 경로를 _**chroot**_ 시킨 디렉토리를 _**system root**_로 사용하겠다는 의미이므로, chroot directory에 기존의 file system과 실행 파일, library 등등이 존재해야 합니다.

다음의 script는 _**/chroot**_ 디렉토리에 chroot 환경을 구성하는 script 입니다. 이 script를 구동하면 일단 chroot 환경에서 동작이 가능 합니다.

```bash
[root@an3 ~]$ cat chroot-installer.sh
#!/bin/bash

# Chroot installer
# Copyright (c) 2016 JoungKyun.Kim <http://oops.org> all rights reserved
# License by BSD 2-Clause

chroot="/chroot"
create_dir="bin dev home lib64 lib proc sbin var/tmp var/log usr"
bind_dir="home var/log usr proc dev"
root_bind="bin sbin lib64 lib"

if [ "${chroot}" = "" -o "${chroot}" = "/" ]; then
    echo "Invalid base chroot path \"${chroot}\"" 1> /dev/stderr
    exit 1
fi


mkdir -p ${chroot}

case "$1" in
    start)
        mount | grep "${chroot}/usr" >& /dev/null
        [ $? -eq 0 ] && echo "Aleady constructed chroot" && exit 1

        if [ ! -d "${chroot}/tmp" ]; then
            mkdir -p ${chroot}/tmp
            chmod 1777 ${chroot}/tmp
        fi

        for cdir in ${create_dir}
        do
            mkdir -p ${chroot}/${cdir} &> /dev/null
        done

        for cbind in ${bind_dir}
        do
            mount --bind /${cbind} ${chroot}/${cbind}
        done
        mount -o bind /dev/pts ${chroot}/dev/pts

        for rbind in ${root_bind}
        do
            mount -o bind /usr/${rbind} ${chroot}/${rbind}
        done

        rsync -av /etc/ ${chroot}/etc/ >& /dev/null
        rm -f ${chroot}/etc/shadow*

        ;;
    *)
        umount ${chroot}/dev/pts
        for cbind in ${root_bind} ${bind_dir}
        do
            umount ${chroot}/${cbind}
        done
        ;;
esac

exit 0
[root@an3 ~]$
[root@an3 ~]$ bash ./chroot-installer.sh start
[root@an3 ~]$ [ -f /chroot/bin/bash -a ! -f "/chroot/etc/shadow" ] && echo "Success" || echo "Failure"
Success
[root@an3 ~]$ bash ./chroot-installer.sh stop
```

chroot 환경을 구성하기 전에 상단의 _**chroot-installer.sh**_를 구동해 주면 _**/chroot**_에 Jail 환경을 구성해 주게 됩니다.

다만, 위의 script는 chroot 시에 동작을 할 수 있는 최소한의 환경이며, 용량 문제로 _**RW**_가 가능한 bind mount를 한 상태이기 때문에 100% Jail 시켰다고 할 수가 없습니다. 이 script를 기본으로 하여 dev, dev/pts, sys, proc를 제외한 나머지는 copy가 되어야지 정말 system jail을 구성했다고 할 수 있으니, 이 script를 기준으로 _**chroot Jail**_ 환경을 구성하시기 바랍니다.

