# 안녕 리눅스 알려진 버그

## 1. jfbterm

 * 한글 입력시에 gabage data가 입력 됨.
 * CentOS 6.7 kernel 부터 발생하는 문제
 * 원래 segfault가 발생하였으며, segfault 발생시에 tty가 freeze 되어 버리는 문제가 있으며, 일단 이 문제의 우선 순위가 낮아서, segfault를 발생시키지 않고 gabage data를 입력하게 하여 TTY가 freeze 되는 문제만 해결 해 놓음
 * 추후 해결해야 할 문제 임