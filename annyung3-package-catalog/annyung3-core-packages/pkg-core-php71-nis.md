# php71-nis

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

* [php71](../annyung3-addon-packages/pkg-addon-php71.md)

## Sub Packages:

* None

## Releated Packages:

* [php-nis](pkg-core-php-nis.md)

