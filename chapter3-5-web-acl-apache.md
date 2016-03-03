# Apache 2.4

apcahe 2.4의 access control에 대해서는 다음 문서를 참조 하십시오.

 * http://httpd.apache.org/docs/2.4/en/howto/access.html
 * http://httpd.apache.org/docs/2.4/en/howto/auth.html

## 1. IP or host based

apache 2.4 부터는 apache 2.2까지 가능했던 아래와 같은 문법을 지원하지 않습니다.

```apache
<Directory /some/path>
  Order deny,allow
  deny from all
  allow from 127.0.0.1
</Directory>
```

