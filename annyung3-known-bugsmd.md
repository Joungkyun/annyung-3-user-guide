# 안녕 리눅스 알려진 버그

## 1. jfbterm

* 한글 입력시에 gabage data가 입력 됨.
* CentOS 6.7 kernel 부터 발생하는 문제
* 원래 automata에서 segfault가 발생하였으며, segfault 발생시에 tty가 freeze 되어 버리는 문제가 있으며, 일단 이 문제의 우선 순위가 낮아서, segfault를 발생시키지 않고 gabage data를 입력하게 하여 TTY가 freeze 되는 문제만 해결 해 놓음
* 추후 해결해야 할 문제 임

## 2. httpd-authn-google

* _**AuthType**_을 _digest_로 설정했을 경우 segfault 발생
* 역시 우선 순위가 낮아서 추후 살펴볼 예정
* _**AuthType**_을 _Basic_으로 사용할 것!

## 3. php-fpm, php56-fpm

* reload\(-USR2 signal\) 시에 process가 죽어 버림.
  * `service php-fpm reload`
  * `systemctl reload php-fpm`

## 4. PHP composer 사용시, "Invalid version string" 에러 발생

* 안녕 리눅스 PHP의 PHP\_VERSION 상수에 "_**AnNyung**_" 문자열이 포함되어 있어 발생하는 문제.
* [_**3.4.9 composer 사용**_](chapter3-4-php.md) 문서를 참조 하여 수정할 수 있음.
* 다음 버전 부터 이 문제가 해결이 되었음





