# Chapter 6. Time Server 운영

1. [Chonry 설정](chapter6-chrony.md)
1. [NTP 설정](chapter6-ntp.md)

<br>

이 장에서는 안녕 리눅스에서 ***Time Server***를 운영하는 방법을 기술 합니다.

***참고:***
>안녕 리눅스 3의 Time server 운영 문서는 ***RHEL 7***과 ***CentOS 7***에서도 동일하게 적용이 가능 합니다. 설정 파일의 위치만 다릅니다. RHEL 7과 CentOS 7에서는 ***/etc/ntp.conf*** 또는 ***/etc/chrony.conf*** 입니다.)

<br>
***Time server***를 하기 위해서는 *ntp* 또는 *chrony*를 이용하여 서비스를 합니다. ***RHEL 7***부터는 *Time service*를 위하여 *ntp* daemon 대신 *chrony* daemon을 기본으로 제공이 되어 지며, ***안녕 리눅스 3*** 역시 *chrony* daemon를 기본으로 제공 합니다.

이 섹션에서는 *ntp*와 *chrony* 모두에 대해서 기술을 합니다.

먼저 Time server에 대한 설정을 하기 전에 필요한 용어에 대해서 먼저 알아 보도록 합니다.

***Reference Clock***
Time server가 참고하고 신뢰할 수 있는 시간

***Strata***
Time server의 계층 구조. 상위 계층 일수록 신뢰도가 높아 집니다.

***stratum 0***
stratum 0은 Time server가 아니라, 시간을 생성하는 장치 입니다. 예를 들어, 원자 시계, 세슘 시계, GPS 같은 장비들을 의미 합니다.

***stratum 1***
stratum 0 장치에 연결이 되어 있는 최상이 Time server

***Resolution***
시간 제공 장치에서 사용하는 시간의 최소 단위를 의미 합니다.

***Precision***
컴퓨터에서 사용하는 시간의 최소 단위를 의미 합니다.

***Accuracy***
시간의 정확성을 의미

***Jitter***
시간 측정 시 발생 하는 오차 중 최소값

***Vander***
시간 측정 시 발생하는 오차 중 최대값



