# Chapter 6. Time Server 운영

이 섹션은 안녕 리눅스에서 ***Time Server***를 운영하는 방법을 기술 합니다.

***Time server***를 하기 위해서는 *ntp* 또는 *chrony*를 이용하여 서비스를 합니다. ***RHEL 7***부터는 *Time service*를 위하여 *ntp* daemon 대신 *chrony* daemon을 기본으로 제공이 되어 지며, ***안녕 리눅스 3*** 역시 *chrony* daemon를 기본으로 제공 합니다.

이 섹션에서는 *ntp*와 *chrony* 모두에 대해서 기술을 합니다.

1. [Chonry 설정](chapter6-chrony)
2. [NTP 설정](chapter6-ntp)