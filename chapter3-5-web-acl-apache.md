# Apache 2.4

apcahe 2.4의 access control에 대해서는 다음 문서를 참조 하십시오.

 * http://httpd.apache.org/docs/2.4/en/howto/access.html
 * http://httpd.apache.org/docs/2.4/en/howto/auth.html

## 1. Deprecated mod_access

apache 2.4 부터는 apache 2.2까지 가능했던 아래와 같은 문법을 기본으로 지원하지 않습니다.

```apache
<Directory /some/path>
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

## 2. IP or Host based access control

