# php56-geoip

### Description:

PHP 5.6 geoip 확장. maxmind에서 제공하는 API와는 다른 버전이다.

### Features:

function base sample codes:

  ```php
<?php
$searches = array ('oops.org', 'kornet.net', 'yahoo.com');

try {
    $g = GeoIP_open (GEOIP_MEMORY_CACHE|GEOIP_CHECK_CACHE);
    if ( GeoIP_db_avail (GEOIP_CITY_EDITION_REV0) )
        $gc = GeoIP_open (GEOIP_CITY_EDITION_REV0, GEOIP_INDEX_CACHE|GEOIP_CHECK_CACHE);

    if ( GeoIP_db_avail (GEOIP_ISP_EDITION) )
        $gi = GeoIP_open (GEOIP_ISP_EDITION, GEOIP_INDEX_CACHE|GEOIP_CHECK_CACHE);

    if ( ! is_resource ($g) )
        exit;

    #echo "TYPE: " . geoip_database_info ($g) ."\n";

    foreach ( $searches as $v ) {
        $r = geoip_id_by_name ($g, $v);
        print_r ($r);

        if ( is_resource ($gc) ) {
            $rc = GeoIP_record_by_name ($gc, $v);
            print_r ($rc);
        }

        if ( is_resource ($gi) ) {
            $ri = GeoIP_org_by_name ($gi, $v);
            echo "    $ri\n";
        }

        #echo "### " . geoip_country_code_by_name ($g, $v) . "\n";
        #echo "### " . geoip_country_name_by_name ($g, $v) . "\n";
    }


    if ( is_resource ($gc) ) GeoIP_close ($gc);
    if ( is_resource ($gi) ) GeoIP_close ($gi);
    GeoIP_close ($g);
} catch ( GeoIPException $e ) {
    fprintf (STDERR, "%s\n", $e->getMessage ());
    $err = preg_split ('/\r?\n/', $e->getTraceAsString ());
    print_r ($err);
}
?>
```

OOP based codes:

  ```php
<?php
$searches = array ('www.example.com', 'oops.org', 'kornet.net', 'yahoo.com');

try {
    $g = new GeoIP (GEOIP_MEMORY_CACHE|GEOIP_CHECK_CACHE);
    if ( GeoIP_db_avail (GEOIP_CITY_EDITION_REV0) )
        $gc = new GeoIP (GEOIP_CITY_EDITION_REV0, GEOIP_INDEX_CACHE|GEOIP_CHECK_CACHE);
    if ( GeoIP_db_avail (GEOIP_ISP_EDITION) )
        $gi = new GeoIP (GEOIP_ISP_EDITION, GEOIP_INDEX_CACHE|GEOIP_CHECK_CACHE);

    #echo "TYPE: " . $g->database_info () ."\n";

    foreach ( $searches as $v ) {
        $r = $g->id_by_name ($v);
        print_r ($r);

        if ( GeoIP_db_avail (GEOIP_CITY_EDITION_REV0) ) {
            $rc = $gc->record_by_name ($v);
            print_r ($rc);
        }

        if ( GeoIP_db_avail (GEOIP_ISP_EDITION) ) {
            $ri = $gi->org_by_name ($v);
            echo "ISP NAME: $ri\n";
        }

        #echo "### " . $g->country_code_by_name ($v) . "\n";
        #echo "### " . $g->country_name_by_name ($v) . "\n";
    }
} catch ( GeoIPException $e ) {
    fprintf (STDERR, "%s\n", $e->getMessage ());
    $err = preg_split ('/\r?\n/', $e->getTraceAsString ());
    print_r ($err);
}
?>
```

### Reference:
* http://svn.oops.org/wsvn/PHP.mod_geoip/trunk/
* http://svn.oops.org/wsvn/PHP.mod_geoip/trunk/sample.php

### Dependencies:
* [GeoIP](pkg-base-GeoIP,md)
* [php56](pkg-addon-php56.md)

### Sub Packages:
* None

### Releated Packages:
* None
