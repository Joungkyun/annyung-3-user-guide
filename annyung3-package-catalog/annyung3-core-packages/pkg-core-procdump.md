# procdump

## Description:

프로세스 심볼 덤프

## Features:

1. 동작하고 있는 process의 실시간 symbol 덤프.
2. 실행 binary가 debug mode로 빌드되었다면 더 자세한 정보가 나온다.
3. php에서 java의 thread dump 하는 것 처럼 사용이 가능

## Reference:

```bash
[root@an3 ~]$ procdump /usr/sbin/memcached 628
/usr/bin/gdb /usr/sbin/memcached
## /usr/sbin/memcached 628 debugging ##

GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-80.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /usr/sbin/memcached...done.
Attaching to program: /usr/sbin/memcached, process 628
Reading symbols from /lib64/libevent-2.0.so.5...Reading symbols from /lib64/libevent-2.0.so.5...(no debugging symbols found)...done.
(no debugging symbols found)...done.
Loaded symbols for /lib64/libevent-2.0.so.5
Reading symbols from /lib64/libpthread.so.0...(no debugging symbols found)...done.
[New LWP 638]
[New LWP 637]
[New LWP 636]
[New LWP 635]
[New LWP 634]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
Loaded symbols for /lib64/libpthread.so.0
Reading symbols from /lib64/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib64/libc.so.6
Reading symbols from /lib64/ld-linux-x86-64.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib64/ld-linux-x86-64.so.2
0x00007ff9dbf807f3 in epoll_wait () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install memcached-1.4.24-1.an3.x86_64
(gdb) #0  0x00007ff9dbf807f3 in epoll_wait () from /lib64/libc.so.6
#1  0x00007ff9dc48b803 in epoll_dispatch () from /lib64/libevent-2.0.so.5
#2  0x00007ff9dc4773ea in event_base_loop () from /lib64/libevent-2.0.so.5
#3  0x00007ff9dc8d7ce9 in main (argc=<optimized out>, argv=<optimized out>) at memcached.c:5724
(gdb) A debugging session is active.

        Inferior 1 [process 628] will be detached.

Quit anyway? (y or n) [answered Y; input not from terminal]
Detaching from program: /usr/sbin/memcached, process 628


[root@an3 ~]$
```

## Dependencies:

* gdb

## Sub Packages:

* None

## Releated Packages:

* None

