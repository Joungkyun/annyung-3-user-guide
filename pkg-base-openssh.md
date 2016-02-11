# openssh

### Description:
An open source implementation of SSH protocol versions 1 and 2

### Changes on AnNyung:
1. X 의존성 제거
2. 서버 기본 설정 변경 (_/etc/ssh/sshd_config_)
 * KeyRegenerationInterval 0
 * UseDNS no
 * GSSAPIAuthentication no
 * X11Forwarding no
 * Banner /etc/issue.net
 * AcceptEnv 에 USER_LANG 환경 변수 추가
3. 클라이언트 기본 설정 변경 (_/etc/ssh/ssh_config_)
 * GSSAPIAuthentication no
 * ForwardX11Trusted no
 * SendEnv 에 USER_LANG 환경 변수 추가
4. Banner 파일(_/etc/issue, /etc/issue.net_)에 Magic Cookie 지원
5. ssh client에서 IDN 지원
6. host key check skip 옵션 추가 (_-H_)

### Sub packages:
* **openssh-clients** - An open source SSH client applications
* **openssh-keycat** - A mls keycat backend for openssh
* **openssh-ldap** - A LDAP support for open source SSH server daemon
* **openssh-server** - An open source SSH server daemon
* **openssh-server-sysvinit** - The SysV initscript to manage the OpenSSH server.
* **pam_ssh_agent_auth** - PAM module for authentication with ssh-agent