# Chapter 6. Time Server 운영

이 장에서는 안녕 리눅스에서 ***Time Server***를 운영하는 방법을 기술 합니다.

***참고:***
>안녕 리눅스 3의 Time server 운영 문서는 ***RHEL 7***과 ***CentOS 7***에서도 동일하게 적용이 가능 합니다. 설정 파일의 위치만 다릅니다. RHEL 7과 CentOS 7에서는 ***/etc/ntp.conf*** 또는 ***/etc/chrony.conf*** 입니다.)

<br>
***Time server***를 하기 위해서는 *ntp* 또는 *chrony*를 이용하여 서비스를 합니다. ***RHEL 7***부터는 *Time service*를 위하여 *ntp* daemon 대신 *chrony* daemon을 기본으로 제공이 되어 지며, ***안녕 리눅스 3*** 역시 *chrony* daemon를 기본으로 제공 합니다.

이 섹션에서는 *ntp*와 *chrony* 모두에 대해서 기술을 합니다.

1. [Chonry 설정](chapter6-chrony.md)
1. [NTP 설정](chapter6-ntp.md)
