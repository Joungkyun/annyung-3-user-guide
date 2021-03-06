# 안녕 리눅스 3 패키지 일람

안녕 리눅스 저장소\(Repository\)는 총 5개의 저장소와 Fedora project의 EPEL repository를 이용하여 운영이 됩니다. 각 저장소에 대한 설명은 다음과 같습니다.

## [\[AN:core\]](annyung3-core-packages/)

* 안녕 리눅스 구성을 위한 패키지
* 안녕 리눅스에서만 제공하는 패키지

## [\[AN:Base\]](annyung3-base-packages/)

* CentOS 7의 패키지를 수정

## [\[AN:xless\]](annyung3-xless-packages.md)

* CentOS 7의 패키지 중 X 의존성 제거한 패키지

## [\[AN:addon\]](annyung3-addon-packages/)

* CentOS / EPEL 에서 지원하지 않는 패키지
* EPEL 패키지를 수정한 패키지

## [\[AN:plus\]](annyung3-plus-packages.md)

* 기본으로 지원하지 않음 \(수동으로 repository 설정을 추가해야 함\)
* 배포본에 있는 버전 외 다른 버전을 지원할 경우
* 현재 안녕 리눅스 3용으로는 배포된 버전이 없음

## [\[EPEL\]](https://fedoraproject.org/wiki/EPEL)

* [https://fedoraproject.org/wiki/EPEL](https://fedoraproject.org/wiki/EPEL)
* 안녕 리눅스에서 직접 관리하는 저장소는 아닙니다.
* Fedora project에서 운영하는 repository로서, RHEL에서 제공하지 않는 pacakge들을 3rd party package 형식으로 지원합니다.
* EPEL 에서 제공하는 epel-release 패키지를 설치 하지 마십시오. elep repository 설정은 annyung-release 패키지에 이미 포함이 되어 있습니다. 또한, a**nnyung-release 패키지에 포함되어 있는 EPEL repository 설정에는 안녕 리눅스의 패키지와 충돌이 발생하는 패키지 목록들을 따로 관리**를 하고 있습니다. 그러므로 epel-release 를 설치할 경우, 충돌이 발생할 수 있습니다.

