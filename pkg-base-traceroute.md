# traceroute

### Description:
Traces the route taken by packets over an IPv4/IPv6 network

### Changes on AnNyung:
1. tcptraceroute 추가
  ```bash
  [root@an3 srpms]$ tcpping -h
  tcpping v1.6 Richard van den Berg <richard@vdberg.org>

  Usage: tcpping [-d] [-c] [-C] [-w sec] [-q num] [-x count] ipaddress [port]

      -d   print timestamp before every result
      -c   print a columned result line
      -C   print in the same format as -C option of fping
      -w   wait time in seconds (defaults to 3)
      -r   repeat every n seconds (defaults to 1)
      -x   repeat n times (defaults to unlimited)

  See also: man tcptraceroute

  [root@an3 srpms]$
  ```
2. tcpping 추가

### Sub packages:
