# lighttpd Access Control


> **목차**
1. IP based access control
2. referer based access control
3. User Agent based access control
4. Country/ISP based access control
5. User based access control
  1. mod_auth
  2. mod_auth_nis


***lighttpd***의 접근 정책은 기본적으로 *mod_access* 에서 제공하는 ***url.access-deny*** 설정을 이용하여, conditonal field를 이용하여 응용을 합니다.

*conditional field*의 종류에 대해서는 [lighty 운영 문서의 *Conditional Configuration* 섹션](http://redmine.lighttpd.net/projects/lighttpd/wiki/Docs_Configuration)을 참조 하십시오.


## 1. IP based access control

$HTTP["remoteip"] block을 이용하여 제어

```php
# allow only 192.168.0.1
$HTTP["remoteip"] == "192.168.0.1" {
  url.access-deny = ("")
}

# allow all ip except 192.168.0.1
$HTTP["remoteip"] != "192.168.0.1" {
  url.access-deny = ("")
}

# allow 192.168.*.* and 121.33.*.*
$HTTP["remoteip"] =~ "^192.168|121.33" {
  url.access-deny = ("")
}
```

위의 방법 외에, 안녕 리눅스는 ***mod_net_access*** 라는 3rd party module을 통하여 좀더 편한 방법을 제공 합니다.

```php
$HTTP["url"] =~ "/server-(status|config)" {
    url.order               = "allow"
    url.list                = ("127.0.0.1", "10.0.0.0/8")
}
```

***mod_net_access*** 모듈에 대해서는 [mod_net_access 문서](http://svn.oops.org/wsvn/Lighttpd.mod_net_access/trunk/README)를 참고 하십시오.

이 기능을 사용하기 위해서는 ***mod_net_access*** 모듈이 load되어야 하며, 위의 예제 처럼 */server-status* 와 */server-config* handler에 사용하기 위해서는, ***mod_net_access*** 모듈이 ***mod_status*** 모듈 보다 먼저 load 되어야 합니다.

```php
server.modules += (
  ...
  "mod_net_access",
  "mod_status",
   ...
)
```

## 2. referer based access control

```php
# access.allow.com 에서의 접근이 아니면 불허
$HTTP["referer"] !~ "^http[s]?://access.allow.com" {
  url.access-deny = ("")
}
```

## 3. User Agent based access control

```php
# Bot 접근 불허
$HTTP["useragent"] =~ "Bot" {
  url.access-deny = ("")
}
```

## 4. Country/ISP based access control

안녕 3의 lighttpd에서는 ***$HTTP["country"]*** 와 ***$HTTP["isp"]*** conditional field를 이용하여 접속자의 country 또는 isp를 이용하여 access control이 가능 합니다.

이 기능은 안녕 리눅스 3에서만 지원을 합니다.

<strong style="color: red;">참고!</strong>  
* 안녕에서 기본 제공하는 krisp database에는 국내의 ISP 정보만 있습니다. (해외 ISP 정보는 들어 있지 않습니다. GeoISP를 이용하여 custom database를 만들 수 있습니다.)
* ***server-status*** 또는 ***server-config*** handler에서 사용하기 위해서는 mod_status 모듈 보다 mod_krisp 모듈이 먼저 load 되어야 합니다.

```php
$HTTP["url"] == "/kor" {
  $HTTP["country"] != "KR" {
      url.access-deny = ("")
  }
}

$HTTP["isp"] != "boradNnet" {
    url.access-deny = ("")
}
```
## 5. User based access control

### 5.1. mod_auth

***apache***의 *htpasswd* file을 사용하여 인증을 하는 방법과 유사합니다. 자세한 사항은 [***mod_auth*** 문서](https://redmine.lighttpd.net/projects/1/wiki/docs_modauth)를 참고 하십시오.

이 기능을 사용하기 위해서는 ***"mod_auth"*** 모듈이 load 되어야 합니다.

```php
server.modules += ("mod_auth")
```

#### 5.1.1. password list 생성

google에서 ***"htpasswd web generator"*** 로 검색을 하면 web상에서 password list를 만들어 주는 tool들을 쉽게 찾을 수 있습니다.

또는, https://redmine.lighttpd.net/projects/1/wiki/docs_modauth 문서를 참고 하십시오.

#### 5.1.2. 인증 설정
```php
$HTTP["url"] == "/req_auth" {
  auth.backend                   = "htpasswd"
  auth.backend.htpasswd.userfile = "/full/path/to/lighttpd-htpasswd.user"
  auth.require                   = (
    "" => (
      "method"  => "basic",
      "realm"   => "Private Area",
      "require" => "user=john|user=smith"
    )
  )
}
```

### 5.2. mod_auth_nis

***mod_auth_nis*** 모듈을 이용하여 NIS 인증이 가능 합니다. ***mod_auth_nis***는 PAM을 이용하지 않고 설정된 NIS entry를 이용하기 때문에 local account는 인증할 수 없습니다.

lighttpd nis module을 사용하기 위해서는, ***NIS*** 구성시에 shadow.byname MAP을 사용하지 않아야 합니다. 이는 passwd 리스트의 password entry의 값이 'x'로 되어 있으면 안된다는 의미입니다. 보안상 상당히 좋지 않기 때문에 사용을 권장하지 않습니다.

자세한 사항에 대해서는 [***mod_auth_nis*** 모듈 문서](http://svn.oops.org/wsvn/Lighttpd.mod_auth_nis/trunk/README)를 참조 하십시오.

이 기능을 사용하기 위해서는 ***"mod_auth"*** 모듈이 load 되어야 합니다.

```php
server.modules += ("mod_auth_nis")
```

```php
$HTTP["url"] == "/req_auth" {
  auth.backend = "nis"
  nis.domain   = "NISDOMAINNAME"
  nis.usermap  = "passwd.byname"
  nis.require  = (
    "" => (
      "realm"    => "Private Area with NIS",
      "require"  => "valid-user"
      #"require" => "user=john|user=smith"
    )
  )
}
```


