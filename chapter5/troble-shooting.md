---
description: bind 시작 시에 발생할 수 있는 로그에 대하여 정리를 한다.
---

# Troble Shooting

안녕 리눅스 3.5 업데이트 시 bind 가 9.9 에서 9.11 로 업데이트가 됩니다. 9.9 의 기존 설정을 이용할 때 발생할 수 있는 로그에 대해서 정리 합니다.

## 5.9.1 the working directory is not writable

> Sep 28 21:17:35 HOSTNAME named\[97531\]: the working directory is not writable

_**/var/named/zone**_ 디렉토리가 named user 에게 write 권한이 있어야 합니다. 다음과 같이 조치 하십시오.

```bash
chown named.root /var/name/zone
```

## 5.9.2 dnssec-lookaside 'auto' is no longer supported

> Sep 28 22:22:37 HOSTNAME named\[99117\]: /etc/named.conf:82: dnssec-lookaside 'auto' is no longer supported

2017년 9월 30일 부터 ISC.org 의 **DNSSEC Lookasize Validation \(DLV\) 서비스**가 중지 되었습니다. 그러므로 더이상 _**dnssec-lookasize**_ 는 기본적으로 사용되지 않습니다. 해당 설정을 제거해 주세요.

## 5.9.3 max-cache-size 문제



> Sep 28 22:21:18 HOSTNAME none:99: 'max-cache-size 90%' - setting to 175921860444MB \(out of 17592186044415MB\)
>
> Sep 28 22:21:18 HOSTNAME named\[99010\]: none:104: Unable to determine amount of physical memory, setting 'max-cache-size' to unlimited

9.11 부터는 /proc/meminfo 로 부터 시스템의 메모리 정보를 가져와서 max-cache-size 를 90%로 자동으로 설정을 하게 되어 있습니다. 안녕의 bind 는 기본으로 chroot 로 구동이 되는데, 이 경우 CHROOT/proc/meminfo 가 없을 경우 발생을 할 수 있습니다.

안녕 리눅스의 bind 를 systemctl 로 실행할 경우에는 이 문제에 대한 사전 처리를 해 주고 있지만 bind 를 직접 구동할 경우에는 위의 문제가 발생할 수 있습니다. 이 경우에는 bind 실행 전에 다음과 같이 문제를 해결할 수 있습니다.

```bash
mkdir -p /var/named/proc
cp -af /proc/meminfo /var/named/proc/meminfo
```

## 5.9.4 error writing NTA file for view ...

> -- Unit named.service has begun shutting down.
>
> 9월 28 22:21:22 HOSTNAME named\[99010\]: received control channel command 'stop'
>
> 9월 28 22:21:22 HOSTNAME named\[99010\]: shutting down: flushing changes
>
> 9월 28 22:21:22 HOSTNAME named\[99010\]: stopping command channel on 127.0.0.1\#953
>
> 9월 28 22:21:22 HOSTNAME named\[99010\]: error writing NTA file for view '\_default': permission denied
>
> 9월 28 22:21:22 HOSTNAME named\[99010\]: error writing NTA file for view '\_bind': permission denied
>
> 9월 28 22:21:22 HOSTNAME named\[99010\]: exiting
>
> 9월 28 22:21:22 HOSTNAME systemd\[1\]: Stopped Berkeley Internet Name Domain \(DNS\).

bind 종료시에 위의 로그와 같이 '\_default' view 에 대한 NTA 파일을 기록하지 못한다는 로그가 남습니다. 이 문제는 다음과 같이 조치할 수 있습니다.

```bash
chown named:root /var/named
```

