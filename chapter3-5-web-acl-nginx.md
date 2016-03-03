# Nginx

> **목차**
1. IP based access control
2. referer based access control
3. User Agent based access control
4. Country/ISP based access control

## 1. IP based access control

```nginx
location / {
  allow 192.168.0.0/24;
  deny all;
}
```

* 값은 IP 주소, CIDR, unix socket, all 을 사용할 수 있습니다.

## 2. referer based access control

***ngx_http_referer module***을 이용하여 제어를 합니다.

```nginx
location /photo/ {
  valid_referers none blocked server_names *.example.com example.* www.example.org/galleries/ ~\.google\.;
  if ( $invalid_referer) {
    return 403;
  }
}
```
* none - referer 없이 direct로 접속하는 경우
* blocked - *Referer* filed가 존재하기는 하지만, 방화벽이나 proxy 서버에 의해서 지워지거나 또는 변조된 경우를 의미합니다. 예를 들어 *http://* 또는 *https://* 로 시작하지 않는 referer 주소를 들 수 있습니다.
* 그 외의 referer 표현법은 [ngx_http_referer_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html) 문서를 참조 하십시오.

## 3. User Agent based access control

***$http_user_agent*** 환경 변수를 이용하여 제어를 합니다.

```nginx
location / {
  # protect Bing bot and Google bot
  if ( $http_user_agent ~* (bing|google) ) {
    return 403;
  }
}
```

## 4. Country/ISP based access control