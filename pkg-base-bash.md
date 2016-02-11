# bash

### Descriptions:
The GNU Bourne Again shell

### Changes on AnNyung:
1. SU_USER 환경 변수가 있을 경우, history에 실행 유저로 SU_USER를 표기

 ```bash
[root@an3 repos]$ history
   14  2016-02-03 16:18:01 oops make a && make s
   15  2016-02-03 16:18:53 oops ls
   16  2016-02-03 16:18:56 oops ./sync
   17  2016-02-03 16:20:12 oops vi /var/log/yum.log
   18  2016-02-03 16:37:22 james rpm -q openssl
   19  2016-02-03 16:38:06 james cd ../SPECS/
   20  2016-02-03 16:38:06 james ls
   21  2016-02-03 16:38:11 oops rpm -q openssh
```

### Sub packages:
 * **bash-doc** - Documentation files for bash
