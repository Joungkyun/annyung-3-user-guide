# php56-chardet

### Description:
PHP 5.6용 libchardet C binding extension

### Features:
  ```php
<?php
if ( ! extension_loaded ('chardet') ) {
    fprintf (STDERR, "%s API isn't init\n", 'chardet');
    exit;
}

$strings = array (
    '안녕하세요 abc는 영어고요, 가나다는 한글 입니다.',
    '안녕',
    '안녕하세요',
    '조금더 길게 적어 봅니다. 어느 정도가 필요할까요? 오호라.. 점점 길어지네',
);

$fp = chardet_open ();

for ( $i=0; $i<count($strings); $i++ ) {
    $moz = chardet_detect ($fp, $strings[$i]);
    print_r ($moz);
}

chardet_close ($fp);
?>
```

### Reference:
* http://svn.oops.org/wsvn/PHP.mod_chardet/trunk/README
* http://svn.oops.org/wsvn/PHP.mod_chardet/trunk/Reference
* http://svn.oops.org/wsvn/PHP.mod_chardet/trunk/sample-oop.php
* http://svn.oops.org/wsvn/PHP.mod_chardet/trunk/sample.php

### Dependencies:
* [php56](pkg-addon-php.md)
* [libchardet](pkg-core-libchardet.md)

### Sub Packages:
* None

### Releated Packages:
* [python-chardet](pkg-core-python-chardet.md)
* php-chardet - PHP chardet extension
