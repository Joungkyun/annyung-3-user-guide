# Nginx

> **목차** 1. IP based access control 2. referer based access control 3. User Agent based access control 4. Country/ISP based access control 5. User based access control

## 1. IP based access control

```text
location / {
  allow 192.168.0.0/24;
  deny all;
}
```

* 값은 IP 주소, CIDR, unix socket, all 을 사용할 수 있습니다.

## 2. referer based access control

_**ngx\_http\_referer module**_을 이용하여 제어를 합니다.

```text
location /photo/ {
  valid_referers none blocked server_names *.example.com example.* www.example.org/galleries/ ~\.google\.;
  if ( $invalid_referer) {
    return 403;
  }
}
```

* none - referer 없이 direct로 접속하는 경우
* blocked - _Referer_ filed가 존재하기는 하지만, 방화벽이나 proxy 서버에 의해서 지워지거나 또는 변조된 경우를 의미합니다. 예를 들어 _http://_ 또는 _https://_ 로 시작하지 않는 referer 주소를 들 수 있습니다.
* 그 외의 referer 표현법은 [ngx\_http\_referer\_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html) 문서를 참조 하십시오.

## 3. User Agent based access control

_**$http\_user\_agent**_ 환경 변수를 이용하여 제어를 합니다.

```text
location / {
  # protect Bing bot and Google bot
  if ( $http_user_agent ~* (bing|google) ) {
    return 403;
  }
}
```

## 4. Country/ISP based access control

Nginx에서의 국가 또는 ISP 제어는 GeoIP와 krisp를 이용하여 가능 합니다.

둘 중, 어느 것을 사용하는냐는 사용자의 선택이지만, 국내의 IP 및 ISP 정보를 취급할 경우에는 krisp를 사용하는 것이 더 정확 합니다.

**참고!**

* GeoIP 모듈은 ISP 정보를 제공하지 않습니다.
* 안녕에서 기본 제공하는 krisp database에는 국내의 ISP 정보만 있습니다. \(해외 ISP 정보는 들어 있지 않습니다. GeoISP를 이용하여 custom database를 만들 수 있습니다.\)
* krisp database는 KISA의 한국 IP database를 기반으로 하여 제작 되어 국내 환경에는 GeoIP보다 정확도가 높습니다.

### 4.1. GeoIP

GeoIP module을 사용하기 위한 자세한 설정은 [nginx ngx\_http\_geoip\_module 문서](http://nginx.org/en/docs/http/ngx_http_geoip_module.html)를 참고 하십시오.

```text
location = /ko/ {
  if ( $geoip_country_code != KR ) {
    return 403;
  }
}
```

### 4.2. krisp

krisp module을 사용하기 위한 자세한 설정은 [nginx ngx\_http\_krisp module 문서](https://github.com/vozlt/nginx-module-krisp/blob/master/README.md)를 참고 하십시오.

```text
location /ko/ {
  if ( $krisp_country_code = KR ) {
    if ( $krisp_isp_code = KORNET ) {
      return 403;
    }
  }
}
```

## 5. User based access control

### 5.1. password list 생성

google에서 _**"htpasswd web generator"**_ 로 검색을 하면 web상에서 password list를 만들어 주는 tool들을 쉽게 찾을 수 있습니다.

또는, [http://nginx.org/en/docs/http/ngx\_http\_auth\_basic\_module.html\#auth\_basic\_user\_file](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html#auth_basic_user_file) 문서를 참고 하십시오.

### 5.2. 인증 설정

```text
localtion / {
  auth_basic "Restricted Content";
  auth_basic_user_file /etc/nginx/.htpasswd;

  if ( $remote_user !~ ^(john|smith|joe)$ ) {
    return 403;
  }
}
```

\*

