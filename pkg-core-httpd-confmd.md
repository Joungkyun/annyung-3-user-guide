# httpd-conf

### Description:

이 패키지는 안녕 리눅스 아파치2의 바이너리를 제외한 설정파일들을
포함하고 있다.


### Dependency:
* None

### Features:

* **/etc/httpd/conf**
  * 기본 설정파일 httpd.conf 포함.
  * httpd.conf는 apache web server가 구동이 가능한 최소한의 설정
  * <u>이 파일은 수정하지 않고, **/etc/httpd/user.d/Default.conf** 수정을 권고</u>
* **/etc/httpd/conf.d**
  * LoadModules.conf (기본 Module loading 설정)
  * 개별 Module 설정 파일 위치
  * Welcome.conf  
    **Document root directory**에 index 파일이 존재하지 않을 경우 기본 index 설정 파일 출력 설정
  * Security.conf
    * _/tmp, /var/tmp, /etc, /usr/local/etc, /var/lib/php_ 의 접근 방지
    * URI에 _tmp, temp, etc, conf, config, log, logs, class, inc, global, install_ 디렉토리가 있으면 접근 방지
    * _.inc, .ph, .template, .conf, .dat_ 파일 확장자 접근 방지
* **/etc/httpd/user.d**
  * Default.conf : 사용자 설정 파일
  * vhost.conf   : 가상 호스트 설정
* **/etc/logrotate.d/httpd**  
  * /var/log/httpd의 로그 중, _*_log, *.log_ 파일을 rotate 시킴
  * rotate 완료 후에, httpd reload 명령이 구동 됨

### Reference:
* None

### Dependencies:
* None

### Sub Packages:
* None

### Releated Packages:
* [httpd](pkg-base-httpd.md) package의 sub package