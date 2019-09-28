# Chapter 6. Time Server 운영

1. [Chonry 설정](chapter6-chrony.md)
2. [NTP 설정](chapter6-ntp.md)

이 장에서는 안녕 리눅스에서 _**Time Server**_를 운영하는 방법을 기술 합니다.

_**참고:**_

> 안녕 리눅스 3의 Time server 운영 문서는 _**RHEL 7**_과 _**CentOS 7**_에서도 동일하게 적용이 가능 합니다. 설정 파일의 위치만 다릅니다. RHEL 7과 CentOS 7에서는 _**/etc/ntp.conf**_ 또는 _**/etc/chrony.conf**_ 입니다.\)

  
 _**Time server**_를 하기 위해서는 _ntp_ 또는 _chrony_를 이용하여 서비스를 합니다. _**RHEL 7**_부터는 _Time service_를 위하여 _ntp_ daemon 대신 _chrony_ daemon을 기본으로 제공이 되어 지며, _**안녕 리눅스 3**_ 역시 _chrony_ daemon를 기본으로 제공 합니다.

이 섹션에서는 _ntp_와 _chrony_ 모두에 대해서 기술을 합니다.

먼저 Time server에 대한 설정을 하기 전에 필요한 용어에 대해서 먼저 알아 보도록 합니다.

_**Reference Clock**_ Time server가 참고하고 신뢰할 수 있는 시간

_**Strata**_ Time server의 계층 구조. 상위 계층 일수록 신뢰도가 높아 집니다.

_**stratum 0**_ stratum 0은 Time server가 아니라, 시간을 생성하는 장치 입니다. 예를 들어, 원자 시계, 세슘 시계, GPS 같은 장비들을 의미 합니다.

_**stratum 1**_ stratum 0 장치에 연결이 되어 있는 최상이 Time server

_**Resolution**_ 시간 제공 장치에서 사용하는 시간의 최소 단위를 의미 합니다.

_**Precision**_ 컴퓨터에서 사용하는 시간의 최소 단위를 의미 합니다.

_**Accuracy**_ 시간의 정확성을 의미

_**Jitter**_ 시간 측정 시 발생 하는 오차 중 최소값

_**Vander**_ 시간 측정 시 발생하는 오차 중 최대값

