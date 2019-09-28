# php56-krisp

## Description:

php 5.6 KRISP 확장

## Features:

sample codes:

```php
<?php
$searches = array ('oops.org', 'kornet.net', 'yahoo.com', 'kldp.org');

try {
    $kr = new KRISP ();

    $kr->mtimeInterval (0);
    $kr->debug ();

    foreach ( $searches as $v ) {
        $r = $kr->search ($v);

        if ( $r === false )
            continue;

        print_r ($r);
        $kr->debug (false);
    }

    $kr->close ();
} catch ( KRISPException $e ) {
    fprintf (STDERR, "%s\n", $e->getMessage ());
    $err = preg_split ('/\r?\n/', $e->getTraceAsString ());
    print_r ($err);
}
?>
```

## Reference:

* [https://github.com/OOPS-ORG-PHP/mod\_krisp/blob/master/sample.php](https://github.com/OOPS-ORG-PHP/mod_krisp/blob/master/sample.php)
* [https://github.com/OOPS-ORG-PHP/mod\_krisp/blob/master/sample-oop.php](https://github.com/OOPS-ORG-PHP/mod_krisp/blob/master/sample-oop.php)

## Dependencies:

* [php56](../annyung3-addon-packages/pkg-addon-php56.md)
* [libkrisp](pkg-core-libkrisp.md)

## Sub Packages:

* None

## Releated Packages:

* [httpd-krisp](pkg-core-httpd-krisp.md)
* [perl-KRISP](pkg-core-perl-krisp.md)
* [php-krisp](pkg-core-php-krisp.md)
* [python-krisp](pkg-core-python-krisp.md)

