# php-nis

## Description:

php NIS 인증 확장

## Features:

sample codes:

```php
<?php
$user = 'test';
$pass = 'testpass';
$domain = 'NIS-DOMAIN';
$map = 'passwd.byname';

if ( nis_auth ($user, $pass, $domain, $map) === FALSE )
    echo "Auth Failed\n";
else
    echo "Auth Success\n";
?>
```

## Reference:

* [https://github.com/OOPS-ORG-PHP/mod\_nis/blob/master/README](https://github.com/OOPS-ORG-PHP/mod_nis/blob/master/README)

## Dependencies:

* [php](../annyung3-base-packages/pkg-base-php.md)

## Sub Packages:

* None

## Releated Packages:

* [php56-nis](pkg-core-php56-nis.md)

