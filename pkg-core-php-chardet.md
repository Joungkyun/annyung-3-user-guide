# php-chardet

### Description:
libchardet C binding php extension

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
* https://github.com/OOPS-ORG-PHP/mod_chardet/blob/master/README.ko.md
* https://github.com/OOPS-ORG-PHP/mod_chardet/blob/master/Reference
* https://github.com/OOPS-ORG-PHP/mod_chardet/blob/master/sample-oop.php
* https://github.com/OOPS-ORG-PHP/mod_chardet/blob/master/sample.php

### Dependencies:
* [php](pkg-base-php.md)
* [libchardet](pkg-core-libchardet.md)

### Sub Packages:
* None

### Releated Packages:
* [python-chardet](pkg-core-python-chardet.md)
* [php56-chardet](pkg-core-php56-chardet.md)
