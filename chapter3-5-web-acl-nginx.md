# Nginx


## 1. IP base access control

```nginx
location / {
  allow 192.168.0.0/24;
  deny all;
}
```

## 2. referer를 이용한 access control

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