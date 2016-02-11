# remount

### Description:

Mount/Unmount remote file system

### Features:
1. autofs와 비슷한 기능이나, 좀더 직관적으로 remote fs를 다루고 싶을 경우 사용
2. systemd를 사용하지 않고, 기존의 init script를 사용한다.

### Reference:
* 구성 파일
 * /etc/rc.d/init.d/remount
 * /etc/sysconfig/remount

```bash
[root@an3 z]$ cat etc/sysconfig/remount
#
# Mount/Unmount Remote Filesystme configuration
#
# Array start count, Don't touch
n      = 1

#
#v[n++] = FILE_SYSTEM_TYPE REMOTE_TARGET LOCAL_MOUNT_TARGET [MOUNT_OPTION]
#
# Use Magic Cookie
#     %{host}    convert full hostname (hostname)
#     %{shost}   convert hostname (hostname -s)
#     %{ghost}   convert hostname (hostname -s | sed 's/-\?[0-9]\+.*$//g)
#

#v[n++] = nfs xxx:/vol/vol1/nfs1 /mnt/nfs1 rw,noatime,nobarrier
#v[n++] = nfs xxx:/vol/vol1/nfs2 /mnt/nfs2 ro
#v[n++] = nfs xxx:/vol/vol1/nfs3 /mnt/nfs3
#v[n++] = ifs xxx:/vol1/test     /mnt/Data rw,noatime,noexec
[root@an3 z]$

```

### Dependencies:
* None

### Sub Packages:
* None

### Releated Packages:
* None
