# authconfig

## Description:

Command line tool for setting up authentication from network services

## Changes on AnNyung:

1. **/etc/pam.d/system-auth-sc**에 nodelay 옵션 추가
2. desktop 파일 제거
3. [ISMS](http://isms.kisa.or.kr/kor/intro/intro01.jsp) 인증 지원
   * 암호 설정 시, 8자 이상, 대문자/소문자/숫자/특수 문자 중 3개의 조건 필요
   * 이전 암호 4개를 기억
   * 5회 로그인 실패 시 120초 간 계정 잠금
4. **authconfig-gtk** 의존성 에러 수정
5. enable-sssd 또는 enable-sssdauth 옵션 사용 시 sssd stop 되는 문제 수정

## Sub packages:

* **authconfig-gtk** - Graphical tool for setting up authentication from network services

