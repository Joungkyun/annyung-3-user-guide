# Chapter 2. Access Control
## 1. 안녕 리눅스 방화벽 설정
### 4. brute force attack 제어

[brute force attack](https://ko.wikipedia.org/wiki/%EB%AC%B4%EC%B0%A8%EB%B3%84_%EB%8C%80%EC%9E%85_%EA%B3%B5%EA%B2%A9)은 우리 말로 **_무차별 대입 공격_** 이라고 번역을 합니다.

사전 공격(Dictionary attack)의 진화된 형태인데, 사전 공격이라는 것은 ID와 password를 일종의 사전화 하여 가지고 있는 데이터로 대입을 하여 인증을 통과하는 공격을 의미합니다.

대표적인 brute force attack의 흔적을 보자면, _/var/log/secure_ 에서 하나의 IP에서 여러 account로 SSH shell login을 시도하는 흔적을 볼 수 있습니다.

```
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

또는, drupal이나 phpBB, word press 등과 같이 전세계적으로 사용이 많이 되는 web application에 spam을 등록하기 위한 bot들이 가입 및 글 등록 시도를 하는 경우를 brute force attack의 대표적인 형태라고 볼 수 있습니다.