# authbind

## Description:

non-root 권한으로 1024 하위의 포트 바인드를 허가하는 유틸리티

## Features:

1. 사용법

   ```bash
   [root@an3 ~]$ authbind --deep command -option command_argument
   ```

2. tomcat을 authbind로 구동

   ```bash
   [root@an3 ~]$ cat /etc/sysconfig/tomcat
   TC_AUTHBIND=yes
   [root@an3 ~]$
   ```

## Reference:

* Search with google

## Dependencies:

* None

## Sub Packages:

* **authbind-devel** - authbind 개발 라이브러리와 헤더 파일
* **authbind-libs** - authbind 동적 라이브러리

## Related Packages:

* None

