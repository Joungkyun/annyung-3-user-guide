# initscrtips

### Description:
inittab 파일과 /etc/init.d 스크립트

### Changes on AnNyung:
 1. USER_LANG 환경 변수가 설정 되면, LANG 환경 변수를 USER_LANG 값으로 변경
  * _/etc/profile.d/lang.sh_
 2. changed sysctl value (/etc/sysctl.d/60-annyung.conf)

  ```bash
[root@an3 ~]$ cat /etc/sysctl.d/60-annyung.conf
kernel.sysrq                               = 0
kernel.msgmnb                              = 65536
kernel.msgmax                              = 65536
kernel.shmmax                              = 68719476736
kernel.shmall                              = 4294967296
kernel.sem                                 = 250 32000 100 4096
vm.zone_reclaim_mode                       = 0
net.ipv4.ip_forward                        = 0
net.ipv4.conf.default.accept_source_route  = 0
net.ipv4.conf.all.rp_filter                = 1
net.ipv4.conf.default.rp_filter            = 1
net.ipv4.conf.all.accept_redirects         = 1
net.ipv4.conf.default.accept_redirects     = 1
net.ipv4.conf.eth0.accept_redirects        = 1
net.ipv4.tcp_fin_timeout                   = 20
net.ipv4.tcp_max_tw_buckets                = 1440000
net.ipv4.tcp_tw_reuse                      = 1
net.ipv4.tcp_syncookies                    = 1
net.ipv4.tcp_max_syn_backlog               = 8192
net.core.netdev_max_backlog                = 8192
net.ipv4.tcp_rmem                          = 4096 25165824 25165824
net.ipv4.tcp_wmem                          = 4096 65536 25165824
net.core.rmem_default                      = 25165824
net.core.rmem_max                          = 25165824
net.core.wmem_default                      = 212992
net.core.wmem_max                          = 25165824
net.core.somaxconn                         = 20000
fs.aio-max-nr                              = 1048576
[root@an3 ~]$
```

 3. patch init wrapper script on /sbin/service
  * init script 검색 대상에 _/sbin/init.d_ 추가

### Sub packages:
* **debugmode** - Scripts for running in debugging mode
