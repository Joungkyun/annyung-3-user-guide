# Apache 2.4

> 목차 1. Deprecated mod\_access 2. 기본 syntax 3. IP or Host based access control 4. User based access control 5. LDAP 인증 6. NIS 인증 7. 국가/ISP based access control 8. Google Authentificator\(Google OTP\) 인증

apahce 2.4의 access control은 apache 2.2의 새로운 인증 모듈을 사용합니다. 기존의 mod\_access는 deprecated 되어 기본으로 사용을 할 수 없습니다.

apache 2.4의 인증 관련 모듈은 인증\(mod\_authn\_XXX\)과 권한\(mod\_authz\_XXX\) 로 나뉘게 됩니다. 여기서는 기본적인 권한 설정과 안녕에서만 제공하는 권한 설정에 대해서 기술을 합니다.

이 외에도 다양한 방법\(ldap, dbm, owner, anonymous 등\)의 권한 설정을 할 수 있으니, apcahe 2.4의 access control에 대해서는 다음 문서를 참조 하시기 바랍니다.

* [http://httpd.apache.org/docs/2.4/en/howto/access.html](http://httpd.apache.org/docs/2.4/en/howto/access.html)
* [http://httpd.apache.org/docs/2.4/en/howto/auth.html](http://httpd.apache.org/docs/2.4/en/howto/auth.html)

## 1. Deprecated mod\_access

apache 2.4 부터는 apache 2.2까지 가능했던 아래와 같은 문법을 기본으로 지원하지 않습니다.

```text
<Directory "/some/path">
  Order deny,allow
  deny from all
  allow from 127.0.0.1
</Directory>
```

apache 2.4에서 위의 문법을 사용하기 위해서는 mod\_access\_compat module을 load 시켜 주어야 합니다. 안녕 리눅스 3에서는 `/etc/httpd/conf.d/LoadModules.conf`에서 설정을 할 수 있습니다.

```bash
[root@an3 ~]$ cat /etc/httpd/conf.d/LoadModules.conf | grep mod_access_compap
#LoadModule access_compat_module    modules/mod_access_compat.so
[root@an3 ~]$
```

`/etc/httpd/conf.d/LoadModules.conf` 에서 mod\_access\_compat.so의 주석을 풀어 주도록 합니다. `mod_access`를 이용한 인증은 인터넷에 많은 문서가 있으며, 또한 apache 2.4부터는 _**deprecated**_ 되어진 syntax이기 때문에 여기서 따로 다루지는 않겠습니다.

## 2. 기본 syntax

apache 2.4의 authorization\(권한\)은 기본을 _**mod\_authz\_core**_ 모듈에서 담당을 합니다. 자세한 사항은 [Apache 2.4 mod\_authz\_core](http://httpd.apache.org/docs/2.4/en/mod/mod_authz_core.html) 모듈 문서를 참고 하십시오.

_**mod\_authz\_core**_ 모듈은 안녕 3의 _**/etc/httpd/conf.d/LoadModules.conf**_에서 기본으로 load 하고 있습니다.

_**Require \[METHOD\] \[VALUES\]**_

* Method \[`all | env | method | expr`\]

```text
  Require all granted      # 모두 허가
  Require all denied       # 모두 거절
  Require method GET       # GET method만 허가
  Require expr "!(%{QUERY_STRING} =~ /secret/)" # query string에 secret 문자가 없을 경우 허가

  SetEnvIf User-Agent ^KnockKnock/2\.0 let_me_in
  Require env let_me_in    # User agent가 KnockKnock/2.0 으로 시작하면 허가
```

## 3. IP or Host based access control

IP 또는 Host 방식의 authorization\(권한\)은 _**mod\_authz\_host**_ 모듈에서 담당을 합니다. 자세한 사항은 [Apache 2.4 mod\_authz\_host](http://httpd.apache.org/docs/2.4/en/mod/mod_authz_host.html) 모듈 문서를 참고 하십시오.

역시 안녕 3에서는 _**LoadModules.conf**_에서 기본으로 load 하고 있습니다.

_**mod\_authz\_host**_ 모듈은 _Require_ 지시자에 _ip_와 _host_ method를 사용가능 하게 합니다.

> ```text
>   Require ip 127.0.0.1 192.16 172.17.10.0/16 10.0.0.0/8
>   Require host .domain.com
>   Require local
> ```

* _**IP**_ method \(_**Require ip ...**_\)
  * IP 주소 \(127.0.0.1\)
  * IP 주소의 일부분 \(192.168\)
  * Network/Netmask \(172.16.0.0/255.255.0.0\)
  * Network/nnn 방식의 CIDR \(172.16.0.0/16\)
  * IPv6 가능
* _**HOST**_ method \(_**Require host ...**_\)
  * hostname \(domain.com\)
  * sub domain \(.domain.com\)
  * _host_ method는 _**HostnameLookups**_ 값이 Off일 경우에는 사용이 불가능 합니다.
* _**LOCAL**_ method \(_**Require local**_\)
  * value 가 없음
  * local에서의 접근만 허가할 경우
  * 127.0.0.0/8
  * ::1
  * client와 host의 IP가 같을 경우

## 4. User based access control

### 4.1. password list file 만들기

google에서 _**"htpasswd web generator"**_ 로 검색을 하면 web상에서 password list를 만들어 주는 tool들을 쉽게 찾을 수 있습니다.

또는, [https://httpd.apache.org/docs/2.2/ko/programs/htpasswd.html](https://httpd.apache.org/docs/2.2/ko/programs/htpasswd.html) 문서를 참고 하십시오.

### 4.2. 설정

사용자 방식의 authorization\(권한\)은 _**mod\_auth\_user**_ 모듈에서 담당합니다. 자세한 사항은 [Apache 2.4 mod\_authz\_user](http://httpd.apache.org/docs/2.4/en/mod/mod_authz_user.html) 모듈 문서를 참고 하십시오.

역시 안녕 3에서는 _**LoadModules.conf**_에서 기본으로 load 하고 있습니다.

_**mod\_authz\_user**_ 모듈은 _Require_ 지시자에 _user_ method와 _valid-user_ method를 제공 합니다.

> ```text
> <Directory "/some/path">
>   AuthType Basic
>   AuthName "Protected Area"
>   AuthUserFile "/usr/local/apache/passwd/passwords"
>   AuthBasicProvider file
>   # permit access only john and david
>   Require user john david
>   # permit all granted user
>   #Require valid-user
> </Directory>
> ```

* _**valid-user**_ method \(_**Require valid-user**_\)
  * 별도의 값을 가지지 않습니다.
  * 인증을 통과한 모든 유저에 대해서 허가를 합니다.
* _**user**_ method \(_**Require user ...**_\)
  * 등록된 유저만 허가 합니다.

## 5. LDAP 인증

apache와 LDAP을 연동하기 위해서는 mod\_ldap package가 필요 합니다.

```bash
[root@an3 ~]$ yum install mod_ldap
```

* LDAP 구성은 [2.3.1 OpenLDAP 인증 통합](https://www.gitbook.com/book/joungkyun/annyung-3-user-guide/edit#/edit/master/chapter2-3-auth-integrate-openldap.md) 문서를 참고 하십시오.
* 다음 구성은 https에서만 인증이 허가 됩니다.
* ldap1과 ldap2 redundency 구성을 합니다.

```text
# context가 server config 이므로 <virtualhost>, <location>, <directory>
# block에 포함되면 안됩니다.
<IfModule ldap_module>
    LDAPTrustedMode SSL
    # LDAP SSL 구성시에 사용한 CA 인증서
    LDAPTrustedGlobalCert CA_BASE64 /etc/pki/startssl/startssl-sub.class2.server.ca.sha2.pem
</IfModule>

# LDAP server redundency 설정
<IfModule authn_core_module>
    <AuthnProviderAlias ldap ldap-ldap1>
        <IfModule authnz_ldap_module>
            AuthLDAPURL ldaps://ldap1.oops.org/ou=People,dc=oops,dc=org?uid?sub?(objectClass=posixAccount)
            AuthLDAPBindDN "uid=ssomanager,ou=admin,dc=oops,dc=org"
            AuthLDAPBindPassword "SSO_MANAGER_평문_암호"
        </IfModule>
    </AuthnProviderAlias>
    <AuthnProviderAlias ldap ldap-ldap2>
        <IfModule authnz_ldap_module>
            AuthLDAPURL ldaps://ldap2.oops.org/ou=People,dc=oops,dc=org?uid?sub?(objectClass=posixAccount)
            AuthLDAPBindDN "uid=ssomanager,ou=admin,dc=kldp,dc=org"
            AuthLDAPBindPassword "SSO_MANAGER_평문_암호"
        </IfModule>
    </AuthnProviderAlias>
</IfModule>

<Location /ldap-test>
    AuthName "LDAP test"
    Authtype Basic

    <IfModule authnz_ldap_module>
        <IfModule ssl_module>
            # https 접근이 아니면 403
            SSLRequireSSL
        </IfModule>

        AuthBasicProvider ldap-ldap1 ldap-ldap2
    </IfModule>

    Require valid-user
    # 10.0.0.0/8 네트워크에서의 접근은 인증 없이 허가
    Require 10.0.0.0/8
</Location>
```

설정을 마쳤으면, ssomanager의 암호가 설정에 있기 때문에 설정한 파일에서 other의 권한을 모두 삭제 합니다.

```bash
[root@an3 ~]$ chmod 640 /etc/httpd/user.d/vhost.conf
```

문법 오류 검사 후, apache를 재시작 합니다.

```bash
[root@an3 ~]$ httpd -t
Syntax OK
[root@an3 ~]$ service httpd restart
```

## 6. NIS 인증

안녕 3의 apache에서 NIS를 이용한 인증 및 권한 설정을 제공 합니다. 이 기능을 사용하기 위해서는 [httpd-nis](../../annyung3-package-catalog/annyung3-core-packages/pkg-core-httpd-nis.md) 모듈이 필요 하며, apache가 동작하는 시스템에 YPBIND가 구동되고 있어야 합니다.

apache nis module을 사용하기 위해서는, _**NIS**_ 구성시에 shadow.byname MAP을 사용하지 않아야 합니다. 이는 passwd 리스트의 password entry의 값이 'x'로 되어 있으면 안된다는 의미입니다. 보안상 상당히 좋지 않기 때문에 사용을 권장하지 않습니다.

만약 _**NIS**_ 구성 하에서 웹 인증이 필요하다면, _**NIS**_를 _**shadow**_로 구성을 하시고, cronjob script를 이용하여 _**.htpasswd**_ 파일을 생성하시는 것을 권장 합니다.

```bash
[root@an3 ~]$ yum install httpd-nis
```

사용 방법은 다음과 같습니다.

```text
<Directory "/some/path">
  AuthType Basic
  AuthName "Authentication With NIS"
  AuthBasicProvider nis

  <IfModule nis_auth_module>
    NisDomain NISDOMAINNAME
    NisPwMap  passwd.byname
  </IfModule>

  Require valid-user
  #Require user john smith
</Directory>
```

## 7. 국가/ISP based access control

안녕 3에서는 _**libkrisp**_ library를 이용하여 국가 또는 ISP로 권한 제어를 할 수 있습니다. 이를 위해서는 [httpd-krisp](../../annyung3-package-catalog/annyung3-core-packages/pkg-core-httpd-krisp.md) 모듈이 필요 합니다.

```bash
[root@an3 ~]$ yum install httpd-krisp
```

자세한 사항에 대해서는 [Apache mod\_krisp 문서](http://svn.oops.org/wsvn/Apache.mod_krisp/trunk/apache2/README)를 참고 하십시오.

_**/etc/httpd/conf.d/krisp.conf**_ 에서 관련 설정을 보실 수 있습니다.

이 모듈을 사용할 경우 apache에 다음의 환경 변수가 생성이 됩니다.

* KRISP\_ORIGINAL\_IP
* KRISP\_CHECK\_IP
* KRISP\_COUNTRY\_CODE
* KRISP\_COUNTRY\_NAME
* KRISP\_ISP\_CODE
* KRISP\_ISP\_NAME

_**krisp**_ module을 이용한 권한 제어는 mod\_authz\_core의 _expr_ method 또는 _Rewrite rule_을 이용하여 가능 합니다.

### 7.1. _**expr**_ method

```text
<Directory "/some/path">
  # KRISP_COUNTRY_CODE 가 RU나 IN이 아니면 허가
  Require expr "!(%{ENV:KRISP_COUNTRY_CODE} =~ /^RU|IN$/)"
  # KRISP_COUNTRY_CODE가 KR이 아니면 허가
  Require expr "%{ENV:KRISP_COUNTRY_CODE} != 'KR'"
</Directory>
```

### 7.2. _**Rewrite Rule**_

```text
<Directory "/some/path">
  RewriteCond     %{HTTP_HOST}          ^domain\.com$
  RewriteCond     %{ENV:KRISP_ISP_CODE} ^BORANET|KINXINC$ [NC]
  # KRISP_ISP_CODE가 BORANET이나 KINXINC면 접근 제한
  RewriteRule     .*                    - [R=403]
</Directory>
```

## 8. Google Authentificator\(Google OTP\) 인증

이 모듈에 대한 자세한 설명은 [공식 홈페이지 문서](https://code.google.com/archive/p/google-authenticator-apache-module/wikis/GoogleAuthenticatorApacheModule.wiki)를 참고 하십시오.

### 8.1 module dependency

Google OTP를 이용하기 위해서는 _**httpd-authn-google**_ package를 설치해야 합니다.

```bash
[root@an3 ~]$ yum install httpd-authn-google
```

또한, apache에 google OTP설정을 하기 위해서는 다음 모듈들에 의존성이 있습니다. \(일단 안녕 3에서는 기본적으로 load가 되고 있으니 신경 쓸 일은 없습니다.\)

* mod\_auth\_basic
* mod\_authn\_core
* mod\_authz\_core
* mod\_authz\_user

### 8.2 configuration file

설정 파일은 _/etc/httpd/conf.d/authn-google.conf_ 에서 Module을 load하고 있으며, 인증 설정은 _/etc/httpd/user.d_ 에서 적당한 위치에 하면 됩니다.

secret file 위치는 _/etc/httpd/ga\_auth_ 에 username으로 저장하면 됩니다.

```bash
[root@an3 ~]$ cat /etc/httpd/conf.d/authn-google.conf
#
# Google Authentificator for Apache
#
# Dependent modules:
#     mod_auth_basic
#     mod_authn_core
#     mod_authz_core
#     mod_authz_user
#
# <Directory /some/path>
#     AuthType Basic
#     AuthName "Private area"
#     AuthBasicProvider "google_authenticator"
#     Require valid-user
#
#     GoogleAuthUserPath ga_auth
#     GoogleAuthCookieLife 3600
#     GoogleAuthEntryWindow 2
# </Directory>
#

Loadmodule authn_google_module modules/mod_authn_google.so

# Local variables:
# tab-width: 4
# c-basic-offset: 4
# End:
# vim: set filetype=apache syntax=apache noet sw=4 ts=4 fdm=marker:
# vim<600: noet sw=4 ts=4:
[root@an3 ~]$
```

### 8.3 Secret file 생성

다음 작업은 root 권한으로 하는 것을 가정으로 설명 합니다.

_**google-authenticator**_ 를 _--secret_ 옵션으로 _/etc/httpd/ga\_auth/@USERNAME@_ 에 저장하도록 실행합니다.

파일 이름이 login ID가 되므로, _**@USERNAME@**_을 사용할 ID로 치환해 주시면 됩니다.

```bash
  [root@an3 ~]$ google-authenticator --secret=/etc/httpd/ga_auth/@USERNAME@ -t -d --label=an3.oops.org --issuer=oops.org -r 3 -R 30
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/an3.oops.org%3Fsecret%3D65SANAUX4QX7OM5F%26issuer%3Doops.org

                  [QR-CODE-IAMGE]

  Your new secret key is: 65SANAUX4QU7OM5F
  Your verification code is 424982
  Your emergency scratch codes are:
    77133342
    84939994
    94216212
    47785087
    28719988

  Do you want me to update your "/etc/httpd/ga_auth/@USERNAME@" file (y/n) y

  By default, tokens are good for 30 seconds and in order to compensate for
  possible time-skew between the client and the server, we allow an extra
  token before and after the current time. If you experience problems with poor
  time synchronization, you can increase the window from its default
  size of 1:30min to about 4min. Do you want to do so (y/n) By default, tokens are good for 30 seconds and in order to compensate for
  possible time-skew between the client and the server, we allow an extra
  token before and after the current time. If you experience problems with poor
  time synchronization, you can increase the window from its default
  size of 1:30min to about 4min. Do you want to do so (y/n) y

  [root@an3 ~]$
```

이렇게 생성을 하면, id와 OPT로 생성된 6자리 verication code만으로 인증이 됩니다. 만약 password 어구까지 해서 2 factor 인증을 하고 싶다면, _secret file_에 다음의 라인을 추가해 주면 됩니다. 라인 시작이 따옴표\("\)로 시작해야 합니다.

```bash
" PASSWORD = @PLAIN_PASSWORD@
```

다음, 이 파일을 apache가 읽을 수 있도록 권한을 부여 합니다.

```bash
[root@an3 ~]$ chown root.nobody /etc/httpd/ga_auth/*
[root@an3 ~]$ chmod 440 /etc/httpd/ga_auth/*
[root@an3 ~]$ cat /etc/httpd/ga_auth/bbuwoo
65SYBUUX4QU7OM5F
" RATE_LIMIT 3 30 1455873786
" WINDOW_SIZE 17
" DISALLOW_REUSE 48529126
" TOTP_AUTH
" PASSWORD = @PLAIN_PASSWORD@
"
77133342
84939994
94216212
47785087
28719988
[root@ane ~]$
```

### 8.4 Client 설정

사용자들에게 우선 OPT program 설치에 대해서 안내를 합니다.

* iPhone이나 Android의 경우에는 App Store에서 _**Google OTP**_를 설치 합니다.
* Smart Phone 이 없거나 PC에서 사용하고 싶은 경우에는 [WinAuth](https://winauth.com/download/)를 이용합니다.

설치 안내와 함께, OTP 등록을 위해서, 위에서 secret 파일을 생성할 때 나온 메시지 중, _**secret key**_를 같이 알려 주도록 합니다. 이 메시지를 확인할 수 없다면 생성된 _**secret file**_의 첫 번째 라인이 _**secret key**_이니 이를 알려 주시면 됩니다. 또한 PASSWORD를 추가 했다면 _password_와 _secret key_를 모두 알려 주어야 합니다.

password를 지정하여 2 factor 인증을 하게 할 경우, 암호는 _password + verication code_ 입니다. 예를 들어:

* password : ehfhtl&vlxj
* valid code : 123 456

일 경우, 암호는 _**"ehfhtl&vlxj123456"**_ 입니다. 공백 문자 없이 암호와 verication code를 붙여서 입력 하면 됩니다.

### 8.4 인증 설정

Google Authentificator Apache module은 _**AuthType**_으로 _Basic_과 _Digest_를 모두 지원 합니다만, 현재 버전에서 Digest 방식은 segfault를 발생 시키고 있습니다. 그러니 _Basic_ type으로 사용하시기 바랍니다.

현재 이 모듈이 공식 사이트에서 2013년 이후 관리되고 있지 않으므로, 이 문제가 해결되기는 쉽지 않을 것으로 보이며, 시간이 나는 데로 문제 해결을 해 볼 예정입니다.

```text
<Directory /some/path>
     AuthType Basic
     # AuthName은 원하는 대로 수정
     AuthName "Private area"
     AuthBasicProvider "google_authenticator"
     # 인증 성공한 모든 유저 login
     # 인증을 성공 하였더라고 특정 유저만 허가하고 싶다면 Requires user를 이용할 것.
     Require valid-user

     # secret file location (/etc/httpd/ga_auth)
     GoogleAuthUserPath ga_auth
     # 인증 유지 기간(초)
     GoogleAuthCookieLife 3600
     GoogleAuthEntryWindow 2
</Directory>
```

_/etc/httpd/user.d/_ 아래에 위와 같이 설정을 한 후에, apache 재시작을 하시면 됩니다. _**.htaccess**_ 에서도 설정이 가능 합니다.

