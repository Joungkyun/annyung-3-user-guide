# Apache 2.4

apahce 2.4의 access control은 apache 2.2의 새로운 인증 모듈을 사용합니다. 기존의 mod_access는 deprecated 되어 기본으로 사용을 할 수 없습니다.

apache 2.4의 인증 관련 모듈은 인증(mod_authn_XXX)과 권한(mod_authz_XXX) 로 나뉘게 됩니다. 여기서는 권한(*authorization*) 관련 사항에 대하여 살펴 보도록 합니다.

또한, 기존의 IP, Host 단위의 authorization보다 다양한 방법의 authorization을 할 수 있으니, apcahe 2.4의 access control에 대해서는 다음 문서를 참조 하시기 바랍니다.

 * http://httpd.apache.org/docs/2.4/en/howto/access.html
 * http://httpd.apache.org/docs/2.4/en/howto/auth.html

## 1. Deprecated mod_access

apache 2.4 부터는 apache 2.2까지 가능했던 아래와 같은 문법을 기본으로 지원하지 않습니다.

```apache
<Directory "/some/path">
  Order deny,allow
  deny from all
  allow from 127.0.0.1
</Directory>
```

apache 2.4에서 위의 문법을 사용하기 위해서는 mod_access_compat module을 load 시켜 주어야 합니다. 안녕 리눅스 3에서는 ```/etc/httpd/conf.d/LoadModules.conf```에서 설정을 할 수 있습니다.

```bash
[root@an3 ~]$ cat /etc/httpd/conf.d/LoadModules.conf | grep mod_access_compap
#LoadModule access_compat_module    modules/mod_access_compat.so
[root@an3 ~]$
```
```/etc/httpd/conf.d/LoadModules.conf``` 에서 mod_access_compat.so의 주석을 풀어 주도록 합니다. ```mod_access```를 이용한 인증은 인터넷에 많은 문서가 있으며, 또한 apache 2.4부터는 ***deprecated*** 되어진 syntax이기 때문에 여기서 따로 다루지는 않겠습니다.

## 2. 기본 syntax

apache 2.4의 authorization(권한)은 기본을 ***mod_authz_core*** 모듈에서 담당을 합니다. 자세한 사항은 [Apache 2.4 mod_authz_core](http://httpd.apache.org/docs/2.4/en/mod/mod_authz_core.html) 모듈 문서를 참고 하십시오.

***mod_authz_core*** 모듈은 안녕 3의 ***/etc/httpd/conf.d/LoadModules.conf***에서 기본으로 load 하고 있습니다.

***Require [METHOD] [VALUES]***

 * Method [```all | env | method | expr```]

```apache
  Require all granted      # 모두 허가
  Require all denied       # 모두 거절
  Require method GET       # GET method만 허가
  Require expr "!(%{QUERY_STRING} =~ /secret/)" # query string에 secret 문자가 없을 경우 허가
  
  SetEnvIf User-Agent ^KnockKnock/2\.0 let_me_in
  Require env let_me_in    # User agent가 KnockKnock/2.0 으로 시작하면 허가
```


## 2. IP or Host based access control

IP 또는 Host 방식의 authorization(권한)은 ***mod_authz_host*** 모듈에서 담당을 합니다. 자세한 사항은 [Apache 2.4 mod_authz_host](http://httpd.apache.org/docs/2.4/en/mod/mod_authz_host.html) 모듈 문서를 참고 하십시오.

역시 안녕 3에서는 ***LoadModules.conf***에서 기본으로 load 하고 있습니다.

***mod_authz_host*** 모듈은 *Require* 지시자에 *ip*와 *host* method를 사용가능 하게 합니다.

 * ***IP*** method (***Require ip ...***)
  * IP 주소 (127.0.0.1)
  * IP 주소의 일부분 (192.168)
  * Network/Netmask (172.16.0.0/255.255.0.0)
  * Network/nnn 방식의 CIDR (172.16.0.0/16)
  * IPv6 가능
 * ***HOST*** method (***Require host ...***)
  * hostname (domain.com)
  * sub domain (.domain.com)
  * *host* method는 <u>***HostnameLookups*** 값이 Off일 경우에는 사용이 불가능</u> 합니다.
 * ***LOCAL*** method (***Require local***)
  * value 가 없음
  * local에서의 접근만 허가할 경우
  * 127.0.0.0/8
  * ::1
  * client와 host의 IP가 같을 경우

```apache
<Directory /some/path>
  Require ip 127.0.0.1 192.16 172.17.10.0/16 10.0.0.0/8
  Require host .domain.com
  Require local
</Directory>
```

## 3. User based access control

사용자 방식의 authorization(권한)은 ***mod_auth_user*** 모듈에서 담당합니다. 자세한 사항은 [Apache 2.4 mod_authz_user](http://httpd.apache.org/docs/2.4/en/mod/mod_authz_user.html) 모듈 문서를 참고 하십시오.