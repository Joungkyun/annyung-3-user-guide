# setup

## Description:

시스템 설정 파일과 셋업 파일들

## Changes on AnNyung:

1. [ISMS](http://isms.kisa.or.kr/kor/intro/intro01.jsp) 인증 정책 적용

   ```bash
   [root@an3 z]$ echo $TMOUT
   600
   [root@an3 z]$
   ```

2. HISTSIZE 를 2000으로 증가
3. HISTTIMEFORAMT 지정

   ```bash
   [root@an3 z]$ echo $HISTTIMEFORMAT
   %F %T
   [root@an3 z]$
   ```

4. interactive shell에서 /etc/profile.d/{color,mysql,yum}.sh 구동

## Sub packages:

