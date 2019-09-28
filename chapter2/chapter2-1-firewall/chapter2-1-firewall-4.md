# brute force attack 제어

[brute force attack](https://ko.wikipedia.org/wiki/%EB%AC%B4%EC%B0%A8%EB%B3%84_%EB%8C%80%EC%9E%85_%EA%B3%B5%EA%B2%A9)은 우리 말로 _**무차별 대입 공격**_ 이라고 번역을 합니다.

사전 공격\(Dictionary attack\)의 진화된 형태인데, 사전 공격이라는 것은 ID와 password를 일종의 사전화 하여 가지고 있는 데이터로 대입을 하여 인증을 통과하는 공격을 의미합니다.

[brute force attack](https://ko.wikipedia.org/wiki/%EB%AC%B4%EC%B0%A8%EB%B3%84_%EB%8C%80%EC%9E%85_%EA%B3%B5%EA%B2%A9)은 보통 두가지 부류로 많이 볼 수 있습니다.

1. SSH/telnet/ftp 등의 shell login service에 대한 공격으로 login shell 권한 획득 목적
2. drupal/phpBB/wordpress 등의 web application에 spam 등록 및 web service를 통한 shell 권한 획득을 위한 목적

여기서, 두번째 web에 대한 brute force attack에 대한 방어는 netfilter의 l7-filter를 이용할 수 있으나, 이 rule 작성을 하려면 고도의 지식이 필요하므로 web에 대한 부분은 웹방화벽 solution에서 처리를 하는 것이 더 효과적입니다.

그러므로 여기서는 첫번째 shell login 권한 획득에 대한 방어에 대해서 기술을 합니다.

서버에서 볼 수 있는 대표적인 brute force attack의 흔적을 보자면, _/var/log/secure_ 에서 하나의 IP에서 여러 account로 SSH shell login을 시도하는 흔적을 볼 수 있습니다.

```text
Feb 18 16:00:01 kill sshd[4320]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:01 kill sshd[4319]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:03 kill sshd[4325]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:05 kill sshd[4325]: Failed password for root from 125.88.177.93 port 16277 ssh2
Feb 18 16:00:10 kill last message repeated 2 times
Feb 18 16:00:10 kill sshd[4326]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:10 kill sshd[4325]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:12 kill sshd[4327]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:14 kill sshd[4327]: Failed password for root from 125.88.177.93 port 48843 ssh2
Feb 18 16:00:18 kill last message repeated 2 times
Feb 18 16:00:18 kill sshd[4328]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:18 kill sshd[4327]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:21 kill sshd[4329]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:23 kill sshd[4329]: Failed password for root from 125.88.177.93 port 21861 ssh2
Feb 18 16:00:27 kill last message repeated 2 times
Feb 18 16:00:27 kill sshd[4330]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:27 kill sshd[4329]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:29 kill sshd[4331]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:31 kill sshd[4331]: Failed password for root from 125.88.177.93 port 52989 ssh2
Feb 18 16:00:36 kill last message repeated 2 times
Feb 18 16:00:36 kill sshd[4332]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:36 kill sshd[4331]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:39 kill sshd[4333]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:41 kill sshd[4333]: Failed password for root from 125.88.177.93 port 29602 ssh2
Feb 18 16:00:46 kill last message repeated 2 times
Feb 18 16:00:46 kill sshd[4334]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:46 kill sshd[4333]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:50 kill sshd[4335]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
Feb 18 16:00:52 kill sshd[4335]: Failed password for root from 125.88.177.93 port 10492 ssh2
Feb 18 16:00:58 kill last message repeated 2 times
Feb 18 16:00:58 kill sshd[4336]: Received disconnect from 125.88.177.93: 11:
Feb 18 16:00:58 kill sshd[4335]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=125.88.177.93  user=root
```

burte force attack의 문제점은 3가지의 큰 문제가 있습니다.

1. 엄청난 시도로 서비스 resource에 부하를 줄 수 있다.
2. logging message가 너무 많이 남아 logging 활용성이 떨어진다.
3. 엄청난 시도로 취약한 암호를 사용하는 계정은 탈취를 당할 수 있다.

일단, 안녕 리눅승 shell login에 대한 brute force attack 방어는 _**/etc/oops-firewall/application.conf**_ 에서 할 수 있습니다.

```bash
  [root@an3 ~]$ cat application.conf
  ##########################################################################
  # Application filtering
  # $Id: application.conf 338 2013-01-04 19:17:13Z oops $
  #
  # 이 파일은 특정 공격이나 scan 등을 막기위한 정형적인 서비스를 제공한다.

  ##########################################################################
  # SSH Brute Force Attack
  ##########################################################################
  #
  # BRUTE FORCE FILTER 는 PORT:SECONDS:HIT 로 설정을 한다. 예를 들어 60:10
  # 으로 설정을 할 경우, 60 초 동안 10 번째 접속이 발생하면 다음 60 초동안
  # 필터 한다는 의미이다.
  #
  # Rule:
  #       BRUTE_FORCE_FILTER    = DEST_PORT:SECOND:HIT
  #       BR_BRUTE_FORCE_FILTER = SOURCE_IP|DEST_IP|DEST_PORT:SECOND:HIT
  #
  #BRUTE_FORCE_FILTER = 22:60:10
  BRUTE_FORCE_FILTER    =
  BR_BRUTE_FORCE_FILTER =

  # BFUTE FORCE FILTER 사용시에 로깅을 할지 안할지를 결정한다.
  #
  BRUTE_FORCE_LOG = false

  ** 하략 **
  [root@an3 ~]$
```

위의 설정 주석과 같이 SSH service에 대한 공격을 막기 위해서는 _**BURTE\_FORCE\_FILTER**_ 설정을 해 줍니다.

```bash
  BRUTE_FORCE_FILTER = 22:60:10
```

위의 설정은 다음의 의미를 가집니다.

> 22번 포트로 60초 동안 10번의 접속을 시도하면 다음 60초 동안 접속을 제한한다.

이 설정 처럼 SSH외의 shell login이 가능한 서비스가 있다면 설정을 해 주는 것이 좋습니다.

```bash
  BURTE_FORCE_FILTER = 21:60:10 22:60:10 23:60:10
```

상기 설정은 FTP\(21\), SSH\(22\), TELNET\(23\) 서비스에 대해 적용을 한 예 입니다. 안녕 리눅스의 shell login은 기본으로 SSH\(22\)만 가능하므로, 만약 FTP나 TELNET과 같은 서비스를 추가 한다면, 여기에 추가해 주시면 되겠습니다.

_**BR\_BRUTE\_FORCE\_FILTER**_는 만약 이 서버가 network bridge로 구성이 되어 있을 경우 bridge device를 통해서 지나가는 packet을 제어하기 위해 사용합니다. bridge device를 구성하지 않았다면\(기본으로는 구성되지 않습니다.\) 신경쓰실 필요 없습니다.

