# l4vip

## Description:

L4 DSR 설정을 위한 loopbakc IP 관리 init script

## Features:

* 설정 파일: /etc/sysconfig/l4vip

  ```bash
  shell> cat /etc/sysconfig/l4vip
  # l4vip Configuration
  #
  # Array start count. Don't edit!
  n=0
  #
  # Format:
  # subnet[n++] = IPADDR:DEVICE?+MTU?
  #
  # For Examples:
  subnet[n++] = 1.1.1.1
  subnet[n++] = 1.1.1.10-25
  subnet[n++] = 1.1.1.111:tun0
  subnet[n++] = 1.1.1.112:tun0+1496
  ```

* 구동

  ```bash
  shell> /sbin/service l4vip [start|stop|status]
  ```

## Reference:

* None

## Dependencies:

* None

## Sub Packages:

* None

## Related Packages:

* None

