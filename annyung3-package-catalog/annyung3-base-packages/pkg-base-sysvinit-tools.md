# sysvinit-tools

## Description:

프로세스와 utmp 관리를 위해 사용하는 도구

## Changes on AnNyung:

1. pidof 에 "**-N**" 옵션 추가 - process 수 반환

   ```bash
   [root@an3 ~]$ pidof php-fpm
   1159 1158 1157
   [root@an3 ~]$ pidof -N php-fpm
   3
   [root@an3 ~]$
   ```

## Sub packages:

