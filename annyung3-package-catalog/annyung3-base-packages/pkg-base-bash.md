# bash

## Descriptions:

The GNU Bourne Again shell

## Changes on AnNyung:

1. HISTTIMEFORMAT 기본 설정

   ```bash
   [root@an3 x86_64]$ echo $HISTTIMEFORMAT
   %F %T
   ```

2. HISTTIMEFORMAT 환경 변수 셋팅 시, history에 SU\_USER 환경변수가 있으면 기록
   * thanks for 장현성 \(hsjang @ gmail.com\)

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

## Sub packages:

* **bash-doc** - Documentation files for bash

