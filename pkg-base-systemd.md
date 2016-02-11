# systemd

### Description:
A System and Service Manager

### Changes on AnNyung:
1. 명령행에서 **SHELL** 환경 변수를 사용할 수 있도록 수정
2. rc.local 구동 안되는 문제 수정 (실행 퍼미션 부여)
3. rc.local에서 /etc/rc.locl.mine 호출
 * bug - 2016/02/22 현재 누락 되었음. 다음 버전 업데이트시에 반영 예정
4. booting 후에 **_tty1_**이 clear 되지 않도록 수정

### Sub packages:
* **systemd-devel** - Development headers for systemd
* **systemd-journal-gateway** - Gateway for serving journal events over the network using HTTP
* **systemd-libs** - systemd libraries
* **systemd-networkd** - System service that manages networks.
* **systemd-python** - Python 2 bindings for systemd
* **systemd-resolved** - Network Name Resolution manager.
* **systemd-sysv** - SysV tools for systemd
* **libgudev1** - Libraries for adding libudev support to applications that use glib
* **libgudev1-devel** - Header files for adding libudev support to applications that use glib