# php-krisp

### Description:

php KRISP 확장

### Features:

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

### Reference:
* http://svn.oops.org/wsvn/PHP.mod_krisp/trunk/sample.php
* http://svn.oops.org/wsvn/PHP.mod_krisp/trunk/sample-oop.php

### Dependencies:
* [php](pkg-base-php.md)
* [libkrisp](pkg-core-libkrisp.md)

### Sub Packages:
* None

### Releated Packages:
* [httpd-krisp](pkg-core-httpd-krisp.md)
* [perl-KRISP](pkg-core-perl-KRISP.md)
* [php-krisp](pkg-core-php-krisp.md)
* [python-krisp](pkg-core-python-krisp.md)
