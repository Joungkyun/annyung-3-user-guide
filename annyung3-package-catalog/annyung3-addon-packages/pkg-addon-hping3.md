# hping3

## Description:

명령행 기반의 TCP/IP 패킷 assembler/analyzer

## Features:

1. tcping Alias

   ```bash
   [root@an3 ~]$ cat /etc/profile.d/hping3.sh
   test -f /usr/bin/tcpping || alias tcping >/dev/null 2>&1 || alias tcping="hping3 -S -p 80"
   [root@an3 ~]$
   ```

## Reference:

* [http://wiki.hping.org/](http://wiki.hping.org/)

## Dependencies:

* tcl

## Sub Packages:

* None

## Related Packages:

* None

