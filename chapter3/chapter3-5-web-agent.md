# Web Monitor agent

안녕 리눅스의 _**check-utils**_ package에는 web server를 application level로 모니터링을 할 수 있는 도구가 포함이 되어 있습니다.

1분 마다 cron job으로 체크를 하게 되며, HTTP response code가 200이 아닐 경우, web daemon을 재시작 시킬 수 있습니다.

## 1. configuration

```bash
[root@an3 ~]$ cat /etc/sysconfig/httpd-monitor
# httpd-monitor configuration

# 모니터링 실행 여부
# cronjob 실생시에 이 값이 no이면 그냥 종료함.
execute=no

# 서비스 fail시에 구동할 init file 이름 (기본값 httpd)
# /sbin/srevice ${initfile} [stop|start] 로 사용됨
# 안녕 3에서는 httpd | lighttpd | nginx 중 하나
# 기본값: httpd
initfile=

# 로그 파일 경로
# 기본값: /var/log/httpd/httpwatch.log
logfile=

# connection timeout
# 기본값: 5
ctimeout=

# read timeout
# 기본값: 5
rtimeout=

# 모니터링 주소
# 기본값: $(hostname)
target=

# 접속시 사용할 HTTP/1.1 Host header 값
target_v11=

# 체크할 URI
# 기본값: /robots.txt
uri=

# 서비스 faile시에 apache process의 gdb dump를 뜰지 여부
# AnNyung 3 httpd-conf 패키지의 /usr/sbin/httpd-dump script 필요
# initfile=httpd 일 경우에만 동작함
gdbdump=no
[root@an3 ~]$
```

## 2. Cronjob

* 매 1분 마다 수행하며, execute=yes 일때만 동작 합니다.
* response code가 200이 아닐 경우, web daemon을 재시작 시킵니다.

```bash
[root@an3 ~]$ cat /etc/cron.d/httpd-monitor
# HTTPd protocol checker

* * * * * root /usr/sbin/httpd-monitor >& /dev/null
[root@an3 ~]$
```

## 3. logging

```bash
[root@an3 ~]$ cat /var/log/httpd/httpwatch.log
[2016.03.05 02:11:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:12:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:13:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:14:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:15:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:16:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:17:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:18:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:19:01] http://localhost/robots.txt "Host: domain.org" return code 200
[2016.03.05 02:20:01] http://localhost/robots.txt "Host: domain.org" return code 200
[root@an3 ~]$
```

