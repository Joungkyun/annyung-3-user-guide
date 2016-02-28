# 안녕 리눅스 알려진 버그

## 1. jfbterm 한글 입력 문제

안녕 리눅스에서 제공하는 jfbterm의 한글 출력에는 문제가 없으나, 한글 입력시에 gabage character가 입력되는 문제가 있습니다.

안녕 2에서도 동일한 문제가 있으며, RHEL 6.6 부터 한글 입력시에 segfault 가 발생하고 있으며, 안녕 3에서는 일단 segfault가 발생하면 TTY가 멈추는 관계로 일단 gabage character를 입력 시켜 TTY를 멈추지 않도록 임시 조치를 해 놓은 상태입니다.

안녕 리눅스 3 release 이후에 이 문제 해결에 resource를 투입할 예정입니다.