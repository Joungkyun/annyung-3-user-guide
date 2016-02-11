# php56-nis

### Description:

php NIS 인증 확장

### Features:

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

### Reference:
* http://svn.oops.org/wsvn/PHP.mod_nis/trunk/README

### Dependencies:
* [php56](pkg-addon-php56.md)

### Sub Packages:
* None

### Releated Packages:
* [php-nis](pkg-core-php-nis.md)
