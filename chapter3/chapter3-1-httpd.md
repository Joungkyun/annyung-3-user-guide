# httpd

> 목차 1. 주의 사항 2. 설정 파일 환경 3. HTTP2 protocol 지원 4. SSL 설정 5. /~user 접근 6. CGI 설정 7. PHP 연동 8. Python/Perl/tomcat 등의 연동 9. apcahe 구동

## 1. 주의 사항

1. 안녕 리눅스의 httpd의 기본 MPM은 _**event**_ 입니다.
   1. 안녕 리눅스는 _**event MPM을 사용**_하는 것을 권장 합니다.  

      _**event**_ MPM은 _**prefork**_와 _**worker**_의 hybrid 방식으로 기존의 _**prefork**_와 _**worker**_보다 성능이 월등히 좋습니다.

   2. _**mod\_php**_\(apache module\)를 사용하기 위해서는 MPM을 _**prefork**_로 변경해야 합니다. 권장하지 않습니다. \(추후에 _**mod\_php7**_은 deprecated 될 수 있습니다.\)
   3. _**mod\_php**_ 보다는 _**php-fpm**_\(FastCGI\) 구성을 권고 합니다. 자세한 사항은 _**PHP**_ 섹션을 참고 하십시오.
   4. _**CGI**_ 설정시, MPM이 _**event**_나 _**worker**_일 경우, _**cgi module**_이 아니라 _**cgid**_ 모듈을 사용해야 합니다.
   5. 이 문서 가이드는 _**event MPM**_으로 운영하는 것을 기준으로 기술 합니다.
2. _**user\_dir**_이 기본 off 로 설정 되어 있습니다. \(기본으로 /~user 접근이 차단 되어 있습니다.\)
3. _**/etc/httpd/conf.d/Security.conf**_ 를 꼭 확인 하십시오.  

   웹 공격에 취약한 접근에 대하여 미리 접근을 차단하고 있습니다. 이 설정은 서비스 운영 시에 꼭 확인을 하시기 바랍니다. 이 설정 때문에 원하는 동작이 되지 않을 수 있습니다. 

## 2. 설정 파일 환경

> 먼저, 종합적인 권고를 우선 말하자면, apache 설정을 추가/변경할 것이 있다면, _**/etc/httpd/user.d/Default.conf**_ 에 하십시오! 절대 _**/etc/httpd/conf/httpd.conf**_는 수정하지 마십시오! 모듈 설정은 _**/etc/httpd.conf.d**_에서 운영에 관한 설정은 _**/etc/httpd/user.d**_ 에서 하십시오.

```bash
[root@an3 httpd]$ tree
.
├── conf
│   ├── httpd.conf
│   └── magic
├── conf.d
│   ├── 00-README
│   ├── LoadModules.conf
│   ├── Security.conf
│   ├── Welcome.conf
│   ├── cgi.conf
│   ├── dav.conf
│   └── userdir.conf
├── logs -> ../../var/log/httpd
├── modules -> ../../usr/lib64/httpd/modules
├── run -> /run/httpd
└── user.d
    ├── Default.conf
    └── vhost.conf
```

안녕의 httpd 설정 파일은 [_**httpd-conf**_](../annyung3-package-catalog/annyung3-core-packages/pkg-core-httpd-conf.md)로 분리 되어 있으며, 위와 같은 구조로 되어 있습니다.

* _**/etc/httpd/conf**_
  * 전통적인 httpd의 설정 경로 입니다.
  * _**httpd.conf**_는 apache를 시작하면 동작이 가능한 최소한의 설정으로 구성이 되어 있습니다.
  * _**httpd.conf**_에 설정 되어 있는 지시자들은 모두 설정값이 _**overwrite**_가 가능한 지시자들 입니다.
  * _**httpd.conf**_를 절대 수정 하지 마십시오!
* _**/etc/httpd/conf.d**_
  * apache module 들에 대한 설정을 가지고 있습니다.
  * _**LoadModules.conf**_
    * httpd 기본 패키지에 포함된 module load 설정 이며, 별 다른 기본 설정이 필요 없는 모듈들이 등록 되어 있습니다.
    * 최소한의 동작만을 위한 module들이 load 되어 있기 때문에, 필요한 모듈들을 주석을 해제하여 load 시켜 주어야 할 수 있습니다.
* _**/etc/httpd/user.d**_
  * 모듈 설정과 관련 없는 사이트 설정들을 포함합니다. 
* _**/etc/httpd/httpd.conf**_ 에서 _**/etc/httpd/conf.d/**_**.conf\***를 include 한 다음 _**/etc/httpd/user.d/**_**.conf\***를 include 하기 때문에 중복되는 지시자 중 _**/etc/httpd/user.d/**_에 설정된 지시자가 최종 반영이 됩니다.

## 3. HTTP2 protocol 지원

안녕 리눅스의 apache는 _**http2**_ protpcol을 지원하기 위해서 2.4.17 이상 버전으로 업그레이드 되었으며, 또한 openssl 1.0.1e에 _**ALPN**_ patch를 추가 하였습니다.

_**/etc/httpd/conf.d/LoadModules.conf**_ 에서 다음 라인을 찾아서 주석을 풀어 준 다음, apache를 재시작 합니다.

```text
LoadModule http2_module     modules/mod_http2.so
```

_**http2**_ protocol은 _**h2**_ 와 _**h2c**_\(over SSL\)로 구분이 됩니다.

_**h2**_ protocol은 _**mod\_http2**_가 로드가 되면 _**/etc/httpd/user.d/vhost.conf**_에서 다음의 설정에 의해 자동으로 구동이 됩니다.

```text
<IfModule http2_module>
    Protocols           h2c HTTP/1.1
</IfModule>
```

_**h2c**_\(over SSL\)은 [_**mod\_ssl**_](../annyung3-package-catalog/annyung3-base-packages/pkg-base-httpd.md) package를 설치 하면 자동으로 동작 합니다.

## 4. SSL 설정

SSL을 설정 하기 위해서는 _**mod\_ssl**_ package 설치가 필요 합니다.

```bash
[root@an3 ~]$ yum install mod_ssl
```

설치를 하면 _**/etc/httpd/conf.d/ssl.conf**_ 가 설치 됩니다. 이 설정 파일에 기본적인 _**Chiper**_ 설정 등이 이미 들어가 있으므로, vhost 설정에 인증서 설정을 하는 것으로 간단히 SSL을 운영할 수 있습니다.

인증서 설정은 기존의 CentOS와 동일하니, 여기서 설명은 생략 합니다.

참고로, 인증서에 문제만 없다면 기본 설정으로 [SSLLabs](https://www.ssllabs.com/)의 SSL 인증 등급을 _**A-**_가 되도록 되어 있습니다. 참고로 _**A+**_가 되기 위해서는 _**Strict-Transport-Security**_ 헤더 설정을 해야 하는데, 이 설정은 서브 도메인을 모두 SSL로 보내도 된다는 의미이기 때문에, 도메인 관련 전 사이트가 SSL이 아니면 안됩니다. 만약 보장 한다면 가상 호스트 block에 다음의 설정을 해 주시면 됩니다.

```text
<VirtualHost *:443>
    ServerAdmin    hostmaster@annyung-smaple.org
    ServerName     annyung-spample.org
    DocumentRoot   /home/httpd/html

    SSLEngine on
    SSLCertificateFile      /etc/pki/httpd/annyung-sample.org.crt
    SSLCertificateKeyFile   /etc/pki/httpd/annyung-sample.org.decrypt.key
    SSLCACertificatePath    /etc/pki/httpd
    SSLCACertificateFile    /etc/pki/httpd/startssl-ca.pem
    SSLCertificateChainFile /etc/pki/httpd/startssl-sub.class2.server.ca.sha2.pem

    <IfModule mod_headers.c>
        Header always set Strict-Transport-Security "max-age=31536000;"
    </IfModule>
</VirtualHost>
```

apache 구동 시에 key에 암호가 걸려 있으면 구동할 때 암호를 물어보게 됩니다. 그러므로 key file은 암호를 제거한 파일로 등록을 하도록 합니다.

```bash
[root@an3 ~]$ openssl rsa -in annyung-sample.org.key -out annyung-sample.org.decrypt.key
```

_**SSLPassPhraseDialog**_ 지시자를 이용하여 암호가 걸린 key 파일을 사용할 수는 있으나, 설정이 번거롭고 또한 암호를 기록한 파일이 있어야 한다는 점에서, 그냥 암호를 제거한 key파일을 사용하는 것이 유용하다고 판단이 되어 집니다. 어떤 key 파일을 사용할 지에 대해서는 사용자가 직접 선택을 하시기 바랍니다.

만약, http2 module이 Load 되어 있다면 _**/etc/httpd/conf.d/ssl.conf**_의 다음 설정에 의해 자동으로 http2 protocol로 동작을 합니다.

```text
# HTTP2 configuration
#
<IfModule http2_module>
        Protocols h2 HTTP/1.1
</IfModule>
```

### 5. /~user 접근

안녕 리눅스는 기본으로 /~user 의 접근이 차단 되어 있습니다. 계정 서비스를 하기 위해서는 _**/etc/httpd/conf.d/userdir.conf**_ 에서 다음 설정을 반영한 다음 apache를 재시작 하십시오.

```text
#
# 사용하기 위해서는 모듈 설정 주석을 해제해야 함
#
LoadModule  userdir_module      modules/mod_userdir.so

<IfModule mod_userdir.c>
    UserDir                     On
</IfModule>
```

## 6. CGI 설정

안녕 리눅스에서 cgi를 사용하기 위해서는 _**/etc/httpd/conf.d/cgi.conf**_ 에서 모듈을 load 해 주셔야 합니다. 기본으로 load 설정이 주석 처리 되어 있습니다.

```text
<IfModule mpm_event_module>
   LoadModule  cgid_module     modules/mod_cgid.so
</IfModule>
```

다음 기본으로 설정 되어 있는 cgi 설정들을 사용하기 위해서는 &lt;ifModule&gt;의 조건을 _**cig\_module**_에서 _**cgid\_module**_로 변경 하십시오.

```text
#<IfModule cgi_module>
<IfModule cgid_module>
        <IfModule alias_module>
                ScriptAlias     /cgi-bin/ /home/httpd/cgi-bin/
        </IfModule>

        <Directory /home/httpd/cgi-bin>
                AllowOverride   None
                Options         ExecCGI
                Require         all granted
        </Directory>

        #
        # .cgi, .pl 파일의 경우 위치와 상관없이 cgi로 실행함
        #
        #<IfModule mime_module>
        #       AddHandler      cgi-script      .cgi .pl
        #</IfModule>
</IfModule>

#<IfModule !cgi_module>
<IfModule !cgid_module>
        <Directory /home/httpd/cgi-bin>
                AllowOverride   None
                Options         None
                Require         all denied
        </Directory>
</IfModule>
```

만약 _**prefork**_ MPM을 선택하였다면 _**cgi\_module**_을 load 해 주시면 됩니다.

```text
<IfModule prefork_module>
   LoadModule  cgi_module     modules/mod_cgi.so
</IfModule>
```

## 7. PHP 연동

안녕 리눅스는 기본으로 PHP7을 지원 합니다. 그리고, PHP5의 호환성을 위하여 PHP56 compat package를 FPM으로 지원을 합니다. 그러므로, PHP7과 PHP56을 모두 사용하기 위해서는 mod\_php를 사용하는 것 보다 php-fpm\(FastCGI\)로 구성 하는 것을 권고 합니다.

만약, PHP 5.3 호환이 필요하다면, /etc/php56.d/{cli,fpm}/php53compatible.ini 설정을 이용하십시오. 안녕 리눅스의 PHP56 package에는 PHP 5.4에서 deprecated 되었거나 또는 제거된 기능들을 사용할 수 있도록 옵션을 제공하고 있습니다. preg 정규식의 _**/e**_ modifier 만 제외하고는 거의 PHP 5.3과 거의 비슷하게 동작 시킬 수 있습니다. 이에 대해서는 [_**PHP 섹션**_](chapter3-4-php.md)의 _**6.6. PHP53 comaptible mode**_ 문서를 참고 하십시오.

자세한 사항은 [_**PHP 섹션**_](chapter3-4-php.md)을 참고 하십시오.

## 8. Python/Perl/tomcat 등의 연동

이 환경에 대해서는 별개의 변경 사항이 없기 때문에 CentOS 기준과 Apache 2.4의 기준으로 설정 하시면 됩니다.

## 9. apache 구동

간단한 apcahe control 방법에 대하여 기술 합니다.

* 부팅 시 apache 시작하도록 설정

  ```bash
  [root@an3 ~]$ service httpd enable
  [root@an3 ~]$ # 또는
  [root@an3 ~]$ ntsysv-systemd
  ```

* 부팅 시 apache 시작 하지 않도록 설정

  ```bash
  [root@an3 ~]$ service httpd disable
  [root@an3 ~]$ # 또는
  [root@an3 ~]$ ntsysv-systemd
  ```

* apache 시작

  ```bash
  [root@an3 ~]$ service httpd start
  ```

* apache 정지

  ```bash
  [root@an3 ~]$ service httpd stop
  ```

* apache 재시작

  ```bash
  [root@an3 ~]$ service httpd restart
  ```

* apache 상태 보기

  ```bash
  [root@an3 ~]$ service httpd status
  ```

