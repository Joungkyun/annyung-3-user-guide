# PHP

> 목차
1. 개요


##1. 개요
안녕 리눅스 3은 CentOS/RHEL과 달리 ***PHP 7***을 기본 제공 합니다. 또한 기존의 ***PHP 5***를 사용하는 환경의 호환을 위하여 ***php56*** package를 제공합니다.

안녕에서 제공하는 ***php56*** package에는 ***PHP 5.3*** 이후 <u>deprecated 되어진 기능들을 사용할 수 있는 PHP 5.3 compatible 기능</u>이 있습니다. 5.3 및 5.4 에서 동작하는 code들은 이 기능을 사용하여 코드 수정 없이 사용하실 수 있습니다. (몇가지 예외가 있으며, 예외에 대해서는 PHP 5.3 comapatible mode에서 기술 합니다.)

***php56*** package는 apache module(libphp5.so)를 지원하지 않습니다.

안녕 리눅스에서 제공하는 web server(apache/httpd, lighttpd, nginx)들은 대부분 ***fastcgi*** protocol을 이용하여 PHP와 연동을 합니다. 또한 apache 2.4의 경우, ***event*** MPM이 정식으로 포함이 되었으며, prefork + worker의 hybird 형식이므로, ***prefork***의 성능과 ***worker***의 안전성을 모두 개선하였으므로, ***event*** MPM 에 ***fastcgi***로 연동 하는 것을 권고 합니다. (안녕 리눅스 apache의 기본 MPM은 ***event*** 입니다.)

또한, PHP7과 PHP5를 동시에 운영하여야 한다면 ***fastscgi*** 연동을 선택할 수 밖에 없습니다.(라고 강요 합니다 ^^)

PHP 7 package는 mod_php package(php-7.0.x-x.an3.x86_64.rpm)을 제공을 하고는 있으나, 아마도 곧 decprecated 될 예정입니다. (빌드 시간이 너무 오래 걸립니다 T.T)



