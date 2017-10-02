#PHP

> 목차
1. 개요
2. package 구성
3. 설정 파일
4. 안녕 리눅스 Epoch
5. 3rd party extension build
6. php 호환 package
7. php-fpm 구동
8. Web server 연동
9. composer 사용

##1. 개요
안녕 리눅스 3은 CentOS/RHEL과 달리 ***PHP 7***을 기본 제공 합니다. 또한 여러 버전의 PHP를 구동할 수 있도록 하기 위하여 FPM SAPI를 이용하는 ***php56***, ***php71***과 같은 호환 패키지를 제공 합니다.

안녕에서 제공하는 ***php56*** package에는 ***PHP 5.3*** 이후 <u>deprecated 되어진 기능들을 사용할 수 있는 PHP 5.3 compatible 기능</u>이 있습니다. 5.3 및 5.4 에서 동작하는 code들은 이 기능을 사용하여 코드 수정 없이 사용하실 수 있습니다. (몇가지 예외가 있으며, 예외에 대해서는 PHP 5.3 comapatible mode에서 기술 합니다.)

***php56***과 ***php71*** package는 apache module(libphp5.so)를 지원하지 않습니다.

안녕 리눅스에서 제공하는 web server(apache/httpd, lighttpd, nginx)들은 대부분 ***fastcgi*** protocol을 이용하여 PHP와 연동을 합니다. 또한 apache 2.4의 경우, ***event*** MPM이 정식으로 포함이 되었으며, prefork + worker의 hybird 형식이므로, ***prefork***의 성능과 ***worker***의 안전성을 모두 개선하였으므로, ***event*** MPM 에 ***fastcgi***로 연동 하는 것을 권고 합니다. (안녕 리눅스 apache의 기본 MPM은 ***event*** 입니다.)

또한, PHP7과 PHP5를 동시에 운영하여야 한다면 ***fastscgi*** 연동을 선택할 수 밖에 없습니다.(라고 강요 합니다 ^^)

PHP 7 package는 mod_php package(php-7.0.x-x.an3.x86_64.rpm)을 제공을 하고는 있으나, 아마도 곧 decprecated 될 예정입니다. (빌드 시간이 너무 오래 걸립니다 T.T)

안녕 리눅스의 PHP package 구성에 대해서는 다음 패키지 일람을 참조 하십시오
 * ***[php](pkg-base-php.md)***
 * ***[php56](pkg-addon-php56.md)***
 * ***[php71](pkg-addon-php71.md)***

##2. package 구성
  1. php 7
    * php-common - php 7 설정 파일
    * php - php 7 apache module (php-fpm을 이용하십시오. 곧 deprecated 시킬 예정)
    * php-cli - php 7 CLI interface
    * php-devel - php 7 모듈을 빌드하기 위한 개발 환경
    * php-fpm - php7 FastCGI 연동을 위한 FPM interface
    * php-fpm-conf - php7 FPM 설정 파일
    * php-extension - PHP shared extension
    * 이외 3rd party module - [php 패키지 일람](pkg-base-php.md)의 ***Sub packages*** 항목 참조
  2. php 5.6
    * Docker 환경 만들기 귀찮아서 만들었습니다.
    * php56-common - php 5.6 설정 파일
    * php56-cli - php 5.6 CLI interface
    * php56-devel - php 5.6 모듈을 빌드하기 위한 개발 환경
    * php56-fpm - php 5.6 FastCGI 연동을 위한 FPM interface
    * php56-fpm-conf - php 5.6 FPM 설정 파일
    * php56-extension - PHP 5.6 shared extension
    * 이외 3rd party module - [php56 패키지 일람](pkg-addon-php56.md)의 ***Sub packages*** 항목 참조

##3. 설정 파일

  여기서는 php-fpm package가 설치 되었다는 가정하에 설명합니다. 아래의 파일들은 ***php-common*** 과 ***php-fpm-common***에 포함이 되어 있습니다.
  
  ```bash
  [root@an3 ~]$ cd /etc/php.d
  [root@an3 ~]$ tree
  .
  ├── apache
  │   └── SHARED.ini
  ├── cli
  │   └── SHARED.ini
  ├── fpm
  │   └── SHARED.ini
  ├── fpm.d
  ├── php-cli.ini
  ├── php-fpm-fcgi.ini
  ├── php-fpm.conf
  └── php.ini
  ```
  
  * ***/etc/php.d/php.ini*** - mod_php(apache module)에서 사용하는 PHP 설정 파일
  * ***/etc/php.d/php-cli.ini*** - php CLI에서 사용하는 설정 파일
  * ***/etc/php.d/php-fpm-fcgi.ini*** - php FPM에서 사용하는 설정 파일
  * ***/etc/php.d/{apache,cli,fpm}*** - 각 interface별 shared extension 설정. 여기서는 대부분 모듈 loading이나 shared extension 별 설정이 들어 있습니다.
  * ***/etc/php.d/php-fpm.conf*** - PHP FPM daemon 설정 파일
  * ___/etc/php.d/fom.d/*.conf___ - php-fpm.conf 에서 이 디렉토리의 "*.conf"를 include 합니다. ***POOL*** 별로 설정을 정리 하고 싶을 경우, POOL-NAME.conf 형식으로 이 디렉토리에 저장 하시면 됩니다.

  ***php56*** package의 경우 path가 ***/etc/php56.d*** 에 위치 하며, 설정 파일 구성은 동일 합니다.
  
##4. 안녕 리눅스 Epoch

다음 사항은 ***[php](pkg-base-php.md)***와 ***[php56](pkg-addon-php56.md)*** package 공통 사항 입니다.

###4.1. exec_dir (PHP_INI_SYSTEM) 기능

이 기능은 php 5.4 이전의 safe_mode_exec_dir 기능을 safe_mode가 아닌 경우에도 사용할 수 있도록  그리고 기능을 확장해서 만들어진 기능입니다. 현재 PHP의 경우 5.4 부터 safe_mode가 없어지면서 이 기능도 같이 없어졌지만, 안녕 리눅스이 PHP에서는 exec_dir으로 이름을 수정하여 제공 합니다.

이 기능이 만들어진 이유는 2004년 경 phpBB highlight bug로 KLDP system이 shell injection을 당하여 서버 전체의 데이터가 삭제된 사건이 있었으며, 그 이후로 방치하기 쉬운 시스템의 보안을 어떻게 향상시킬수 있을까 고민하면서 만들어진 기능 입니다. (https://kldp.org/node/45576 참조)

즉, 이 기능이 만들어진 이유는 PHP의 shell injection 공격을 원천적으로 방지하기 위한 목적입니다. 대부분의 shell injection hole은 이제는 무지가 아니라 거의 코딩 실수에 의하여 발생합니다. 이 패치는 이런 코딩 실수에 의한 injection hole을 원천적으로 방지해 줍니다.

또한, 이 기능 때문에 코드가 수정될 필요도 없습니다. 단지 system 계열 함수에서 사용하는 명령어만 exec_dir에 soft link 해 주시면 됩니다.

이 기능의 영향을 받는 function들은 다음과 같습니다. php의 shell_EXEC

  1. 내부적으로 php_exec API를 사용하는 모든 system fuction  
     exec, system, passthru, shell_exec
  2. pcntl_exec
  3. popen
  4. proc_open
  5. backtick operator

다음은 ***exec_dir***의 자세한 기능에 대한 기술 입니다.

  * PHP의 shell injection을 engine level에서 방어하기 위한 기능. engine level에서 처리를 하기 때문에, php code 쪽에서는 아무런 영향이 없음.
  * 2005년 부터 KLDP와 게임 포털 N사 상거래 T사의 core system에 적용되어 검증
  * PHP 5.4 이전의 ***safe_mode_exec_dir***을 ***safe_mode***가 아닌 경우에도 사용할 수 있도록 수정
  * 기본값 ***/var/lib/php/bin***
    * 안녕의 PHP 에서 system 함수를 사용하려면 사용하려면 command가 /var/lib/php/bin 에 soft link나 복사 되어야 함.
    * ***[php56](pkg-addon-php56.md)*** package의 경우에는 ***/var/lib/php56/bin***
  * system function에 의해서 실행된 command의 경로를 강제로 ***exec_dir***로 지정한 경로로 변경
  
  ```php
  // 명령어에 path가 있을 경우 path 치환
  system ('/some/path/command -option cmd_arg');       // 실제 코드
  system ('/var/lib/php/bin/command -option cmd_arg'); // PHP 내부에서 치환되어 수행되는 코드
  
  // 명령어에 path가 없을 경우 path를 attach
  system ('command -option cmd_arg');                  // 실제 코드
  system ('/var/lib/php/bin/command -option cmd_arg'); // PHP 내부에서 치환되어 수행되는 코드
  ```
  * command 치환 parser 재작성 및 기능 개선
    * semi-colon(;), pipe(|), AND/OR 연산(&&, ||), $(), backtick operator(``)에 의한 command 실행에 대한 치환 지원
    * 치환 예제
  
    ```bash
      command; command              => /var/lib/php/command; /var/lib/php/command
      command $(command)            => /var/lib/php/command $(/var/lib/php/command)
      command $(command $(command)) => /var/lib/php/command $(/var/lib/php/command $(/var/lib/php/command))
      command $(command `command`)  => /var/lib/php/command $(/var/lib/php/command `/var/lib/php/command`)
      command `command`             => /var/lib/php/command `/var/lib/php/command`
      command | command             => /var/lib/php/command | /var/lib/php/command
      command && command            => /var/lib/php/command && /var/lib/php/command
      command || command            => /var/lib/php/command || /var/lib/php/command
    ```
    * 주의  
      ***find*** 명령처럼 argument option에서 command를 수행하는 경우는 처리하지 못함.
    
    ```bash
      system ('find . -type -f -exec rm -f {} \;');                  // 실제 코드
      system ('/var/lib/php/bin/find . -type -f -exec rm -f {} \;'); // PHP 내부에서 치환되어 수행되는 코드
    ```
    
***exec_dir*** patch는 https://github.com/OOPS-ORG-PHP/mod_execdir/tree/master/patches 에서 유지보수가 됩니다.

###4.2. disable_functions 기본 적용

  * ***phpinfo***, ***php_uname***, ***sys_get_temp_dir***, ***phpversion***, ***ini_get***, ***ini_set***, ***ini_get_all***, ***get_cfg_var***
  * 이 function들은 php shell들이 사용하는 필수 function드로서 이 function들을 막아서 php shell이 정상 작동하지 못하도록 합니다.
  * 이 function들의 경우 function name 앞에 prefix로 under bar 3개를 붙여주면 사용할 수 있도록 alias function을 제공 합니다.
  ```php
  ___phpinfo ();
  ___php_uname ();
  ___sys_get_temp_dir ();
  ___phpversion ();
  ___ini_get ();
  ___ini_set ();
  ___ini_get_all ();
  ___get_cfg_var ();
  ___assert ();
  ```

  만약, underbar prefix가 붙은 코드를 만들고 안녕이 아닌 PHP에서 구동시 문제가 될 것을 염려 한다면, 다음과 같은 wrapper를 만들어서 include 해 주시면 됩니다. ***php-pear*** package를 설치 하셨다면, ***/usr/share/php/pear/AliasFunc.php*** 파일이 있으니 이 파일을 참고 하시면 되겠습니다.
  
  ```php
  <?php
  if ( ! function_exists ('___ini_get') ) {
      function ___ini_get ($varname) {
          return ini_get ($varname);
      }
  }
  ?>
  ```
  
###4.3. allow_include_extension 기능 추가

  ***php-fpm***의 security.limit_extensions 와 동일한 기능입니다. PHP가 opcode compile 전에 여기에 지정된 확장자가 아니면 compile을 하지 않도록 합니다.
  
  이 기능은 image header에 php code를 숨겨 놓고, code를 injection해서 image header에 숨겨진 php code를 수행하는 것을 방지 할 수 있습니다.
  
  ```php
  <?php
  include '/path/upload/attack-sample.jpg';
  ?>
  ```
  
  위와 같은 공격을 방어할 수 있습니다.
  
  이 기능에 의하여, 허가되지 않은 확장자 파일을 수행하거나 또는 include/require 할 경우 다음과 같은 에러가 발생합니다.
  
  ```php
  <?php
  require_once ('/path/ss.jpg');
  ?>
  ```
  ```bash
  PHP Fatal error:  require_once(): Failed opening required '/path/ss.jpg' security issues in /path/z.php on line 2
  ```

###4.4. file upload시 image header의 injection code 여부 검사 기능 추가

php.ini에서 이 기능에 대한 옵션은 다음과 같습니다.

  * upload_image_check
    * 기본값 On
    * 업로드 이미지 헤더에 php code가 포함되었는지 검사
    * php code 발견 시에 E_WARNING 발생
  * upload_image_check_log
    * 기본값 On
    * 업로드 이미지 헤더 검사 관련 로그 기록
    * php error log가 기록되는 로그 파일에 기록 됨
  * upload_image_check_test
    * 기본값 On
    * 검사만 하고, 에러 레벨만 리턴(E_WARNING 발생 안함)
    * 검사 결과 감지가 되면 UPLOAD_ERR_SEC 반환
    ```php
    <?php
    swtich ( $_FILES['userfile']['error'][0] ) {
        ... 생략 ...
        case UPLOAD_ERR_SEC :
           $errmsg = 'Detection attacking code in image header';
           break;
        ... 생략 ...
    }
    ?>
    ```
  * upload_image_check_whitelist
    * 이미지 헤더 문자열 white list

  * allow_include_extension 에 등록된 확장자 파일을 업로드 할 경우 ***UPLOAD_ERR_ILL*** 에러 코드를 반환함.
    ```php
    <?php
    swtich ( $_FILES['userfile']['error'][0] ) {
        ... 생략 ...
        case UPLOAD_ERR_ILL :
           $errmsg = 'Detection uploading PHP execution file';
           break;
        ... 생략 ...
    }
    ?>
    ```

안녕 리눅스의 PHP를 이용하여 파일 업로드시의 체크는 아래의 코드를 이용할 수 있다.

```php
<?php
function upload_error_check ($upload_error) {
    switch ( $_FILES['userfile']['error'][0] ) {
        case UPLOAD_ERR_INI_SIZE :
            $errmsg = 'The uploaded file exceeds the upload_max_filesize directive in php.ini.';
            break;
        case UPLOAD_ERR_FORM_SIZE :
            $errmsg = 'The uploaded file exceeds the MAX_FILE_SIZE directive that was specified  in the HTML form.';
            break;
        case UPLOAD_ERR_PARTIAL :
            $errmsg = 'The uploaded file was only partially uploaded.';
            break;
        case UPLOAD_ERR_NO_FILE :
            $errmsg = 'No file was uploaded.';
            break;
        case UPLOAD_ERR_NO_TMP_DIR :
            $errmsg = 'Missing a temporary folder.';
            break;
        case UPLOAD_ERR_CANT_WRITE :
            $errmsg = 'Failed to write file to disk.';
            break;
        case UPLOAD_ERR_EXTENSION :
            $errmsg = 'PHP extension stopped the file upload. PHP does not provide a way to ascertain which extension caused the file upload to stop';
            break;
        case UPLOAD_ERR_SEC :
            $errmsg = 'Detection attacking code in image header';
            break;
        case UPLOAD_ERR_ILL :
            $errmsg = 'Detection uploading PHP execution file';
            break;
        default :
            $errmsg = null;
    }
    return $errmsg;
}

foreach ( $_FILES['userfile']['error'] => $upload_error_code ) {
    if ( ($err = upload_error_cehck ($upload_error_code)) != null ) {
        printf ("Upload Failed : %s\n", $err);
        exit;
    }
}

?>
```

###4.5. allow_url_fopen과 allow_url_include를 PHP_INI_ALL로 수정

  * 기본값 Off
  * ***ini_set** 을 이용하여 php code에서 수정할 수 있음.

***allow_url_fopen***과 ***allow_url_include***는 외부의 code를 injection 시키거나 또는 php shell에서 외부 코드를 실행하는데 많이 사용이 되어 집니다.

안녕 리눅스에서는 이 옵션 값을 기본으로 off 시키고, php code에서 필요할 때만 호출을 하여 사용하면 됩니다.

참고:  
***allow_url_fopen***과 ***allow_url_include***을 ***ini_set***으로 활성화 할 때 ***disable_function*** 때문에 ***ini_set***을 사용하지 못할 수 있습니다. 그러므로 ***___ini_set***을 이용하십시오.

###4.6. short_open_tag

***/etc/php.d/php.ini***의 short_open_tag값이 Off 입니다. ***&lt;?*** 대신 ***&lt;?php***를 사용 하십시오.

PHP 5.4 부터는 ***short_open_tag***가 off 이더라도 ***&lt;?=$var&gt;*** 출력이 가능 합니다.


###4.7. realpath_cache_force

PHP는 open_basedir 이 설정 되어 있을 경우, [soft link를 이용한 race condition을 이용하여 open_basedir을 무력화 시키는 버그](http://www.hardened-php.net/advisory_082006.132.html) 때문에, open_basedir이 설정 되어 있을 경우, realpath_cache를 하지 않도록 변경을 하였습니다. 이 이유로, php의 opcache 특성상 항상 파일의 mtime 체크하기 때문에 open_basedir을 사용하면 성능이 굉장히 많이 저하 됩니다.

안녕 리눅스에서는 이 성능 문제를 해결하기 위하여 ***realpath_cache_force*** 옵션을 제공 합니다. 기본값은 off 입니다. php.ini 에서

```ini
realpath_cache_force = On
```

설정을 해 주면, realpath_cache를 하는 대신, 보안적인 문제가 있는 ***link()*** fucntion과 ***symlink()*** function을 사용하지 못하도록 합니다.


##5. 3rd party extension build

간혹, 안녕에서 제공하지 않는 php extension이나 pecl 또는 다른 3rd party extension이 필요한 경우가 있을 수 있습니다. 여기서는 안녕의 PHP에 다른 extension을 지원하도록 하는 방법을 기술 합니다.

PHP는 ***phpize*** (***php56*** package는 ***phpize56***) 명령을 이용하여 shared extension을 만들 수 있습니다.

3rd party entension 또는 php 소스의 ext directorpy에 있는 extension 소스를 확보 합니다. 다음,
  * phpize
  * ./configure
  * make insatll

의 과정으로 설치를 합니다. 설치 후, ***/etc/php.d/{apache,cli,fpm}/SHARED.ini 에서 해당 모듈을 등록해 주면 됩니다.

```bash
[root@an3 ~]$ cd php-7.0.3/ext/pdo_firebird
[root@an3 pdo_firebird]$ phpzie
[root@an3 pdo_firebird]$ ./configure
[root@an3 pdo_firebird]$ make install
```

설치를 한 후, ***/usr/lib64/php/extensions*** 에 해당 so file이 있는지 확인을 해 봅니다. (여기서는 pdo_firebird.so 입니다.) ***php56*** package의 경우에는 ***phpize*** 대신 ***phpize56***을 이용하며,  ***/usr/lib64/php56/extensions*** 에 설치가 됩니다.

```bash
# PHP 7 모듈의 경우 (phpize로 build)
[root@an3 pdo_firebird]$ echo "extension = pdo_firebird.so" >> /etc/php.d/{apache,cli,fpm}/SHARED.ini
# PHP 5.6 모듈의 경우 (phpize56 으로 build)
[root@an3 pdo_firebird]$ echo "extension = pdo_firebird.so" >> /etc/php56.d/{apache,cli,fpm}/SHARED.ini
```

파일이 설치된 것을 확인 했다면 위의 명령으로 module을 load 합니다.


##6. php compatible package

안녕 리눅스는 PHP 버전을 7로 올리면서, 여러 버전의 PHP를 구동할 수 있도록 호환 패키지를 제공 합니다. 현재 제공되는 버전은 다음과 같습니다.

 * ***[php56](pkg-addon-php56.md)***
 * ***[php71](pkg-addon-php71.md)***
 
PHP 호환 패키지들은 apache module(***mod_php***)은 지원하지 않고 ***CLI***와 ***FPM*** SAPI만 제공을 합니다.
 
안녕 리눅스는 PHP 버전을 7로 올리면서 기존의 php5 사용자들의 코드 호환을 위하여 ***[php56](pkg-addon-php56.md)*** package를 제공합니다.

PHP 호환 패키지는 구동 방법, 설정파일 위치, 헤더파일 위치, 명령어 이름, 프로세스 이름, temporary 위치만 제외하고는 모든 것이 ***[php 7](pkg-base-php.md)*** package와 동일한 특성을 가지고 있습니다.

###6.1. 설정 파일 위치

***[php56](pkg-addon-php56.md)*** package의 설정 파일은 ***/etc/php56.d*** 에 있으며, 다음 패키지에 포함되어 있습니다.

  1. php56-common
    * /etc/php56.d/php-cli.ini
    * /etc/php56.d/cli/SHARED.ini
  2. php56-fpm-conf
    * /etc/php56.d/php-fpm-fcgi.ini
    * /etc/php56.d/fpm/SHARED.ini
    * /etc/php56.d/php-fpm.conf
    * /etc/php56.d/fpm.d

특성에 대해서는 ***"3. 설정 파일"*** 섹션을 참고 하십시오.


###6.2. 헤더 파일 위치

***[php56](pkg-addon-php56.md)*** package의 header file들은 ***/usr/include/php56***에 있으며,
이 파일들은 ***php56-devel*** package에 포함되어 있습니다.

###6.3. temporary 위치

***[php56](pkg-addon-php56.md)*** package의 temporary directory 는 다음과 같습니다.

```php
sys_temp_dir      = /var/lib/php56/tmp
exec_dir          = /var/lib/php56/bin
session.save_path = /var/lib/php56/sessions
```

###6.4. Share extension 위치

***[php56](pkg-addon-php56.md)*** package의 shared extension은 ***/usr/lib64/php56/extensions***에 위치 합니다.

###6.5. 명령어 비교

| php 7 | php56 | package |
| :---: | :---: | :---: |
| /usr/bin/php | /usr/bin/php56 | php-cli / php56-cli |
| /usr/sbin/php-fpm | /usr/sbin/php56-fpm | php-fpm / php56-fpm |
| /usr/bin/phpize | /usr/bin/phpize56 | php-devel / php56-devel |
| /usr/bin/php-config | /usr/bin/php56-config | php-devel / php56-devel |

###6.6. PHP53 comaptible mode

안녕 리눅스의 ***[php56](pkg-addon-php56.md)*** package에는 PHP 5.4에서 제거 되었거나 _deprecated_ 되어진 기능들을 사용할 수 있도록 패치가 되어 있습니다. <u>***[php](pkg-base-php.md)***와 ***[php71](pkg-addon-php71.md)*** package에서는 지원하지 않습니다.</u>

PHP 5.3이나 5.4에서 호환성 때문에 5.6으로 업그레이드가 어려운 경우에 이 mode를 사용해서 해결을 할 수 있습니다.

  * /etc/php56.d/cli/php56comaptible.ini
  * /etc/php56.d/fpm/php56compatible.ini
```ini
    ; PHP 5.3 호환 모드 지원
    ;
    ; On으로 설정시 php 5.4에서 제거되었거나 변경된 기능을 5.3과 같이 동작함
    ;
    ; . allow_call_time_pass_reference 지시자 사용 가능 (Default: Off)
    ; . magic_quotes_gpc, magic_quotes_runtime, magic_quotes_sybase 지시자
    ;   및 magic_quotes 관련 함수 사용 가능 (기본값: Off)
    ; . NULL, false, 빈문자열의 값을 가진 변수에 object property를 추가할
    ;   경우에도 E_WARNING 에러 메시지 발생 하지 않음
    ; . TZ 환경 변수로 timezone 지정 가능
    ; . array_combine() 함수에서 key array가 비었을 경우 false 반환
    ; . 5.4에서 제거된 다음의 함수 사용 가능 (E_DEPRECATED level 에러 처리)
    ;   session_is_registered(), session_register(), session_unregister()
    ;   mysqli_bind_param(), mysqli_bind_result(), mysqli_client_encoding(),
    ;   mysqli_fetch(), mysqli_param_count(), mysqli_get_metadata(),
    ;   mysqli_send_long_data(), mysqli::client_encoding()
    ;
    ; Default Value: Off
    ; Development Value: Off
    ; Production Value: Off
    ;
    php53_compatible = Off

    ; This directive allows you to enable and disable warnings which PHP will issue
    ; if you pass a value by reference at function call time. Passing values by
    ; reference at function call time is a deprecated feature which will be removed
    ; from PHP at some point in the near future. The acceptable method for passing a
    ; value by reference to a function is by declaring the reference in the functions
    ; definition, not at call time. This directive does not disable this feature, it
    ; only determines whether PHP will warn you about it or not. These warnings
    ; should enabled in development environments only.
    ; Default Value: On (Suppress warnings)
    ; Development Value: Off (Issue warnings)
    ; Production Value: Off (Issue warnings)
    ; http://php.net/allow-call-time-pass-reference
    ;
    ; This directive has dependency with 'php53_compatible=On'
    allow_call_time_pass_reference = Off

    ; Magic quotes are a preprocessing feature of PHP where PHP will attempt to
    ; escape any character sequences in GET, POST, COOKIE and ENV data which might
    ; otherwise corrupt data being placed in resources such as databases before
    ; making that data available to you. Because of character encoding issues and
    ; non-standard SQL implementations across many databases, it's not currently
    ; possible for this feature to be 100% accurate. PHP's default behavior is to
    ; enable the feature. We strongly recommend you use the escaping mechanisms
    ; designed specifically for the database your using instead of relying on this
    ; feature. Also note, this feature has been deprecated as of PHP 5.3.0
    ; Default Value: On
    ; Development Value: Off
    ; Production Value: Off
    ; http://php.net/magic-quotes-gpc
    ;
    ; This directive has dependency with 'php53_compatible=On'
    magic_quotes_gpc = Off

    ; Magic quotes for runtime-generated data, e.g. data from SQL, from exec(), etc.
    ; http://php.net/magic-quotes-runtime
    ;
    ; This directive has dependency with 'php53_compatible=On'
    magic_quotes_runtime = Off

    ; Use Sybase-style magic quotes (escape ' with '' instead of \').
    ; http://php.net/magic-quotes-sybase
    ;
    ; This directive has dependency with 'php53_compatible=On'
    magic_quotes_sybase = Off
```


##7. php-fpm 구동

***php-fpm*** 은 ***FastCGI Process Manager*** 이며 PHP FastCGI의 기능을 개선하고자 시작된 프로젝트 입니다.
***daemon***으로 동작을 하며 다른 language보다 PHP에서 FastCGI 구현을 쉽게 해 줍니다.

***php-fpm***을 사용하기 위해서는 ***php-fpm*** package 또는 ***php56-fpm*** package가 필요 합니다. 이 둘의 차이는 PHP version이 다르며 ***php-fpm*** package는 7, ***php56-fpm***은 PHP 5.6 기반에서 동작을 합니다.

### 7.1. 안녕 리눅스 php-fpm 안내

***php-fpm***의 설정은 다음의 위치에서 이루어 집니다.

1. php7
  * /etc/php.d/php-fpm-fcgi.ini
  * /etc/php.d/fpm/*.ini
  * /etc/php.d/php-fpm.conf
  * /etc/php.d/fpm.d/*.conf
2. php56
  * /etc/php56.d/php-fpm-fcgi.ini
  * /etc/php56.d/fpm/*.ini
  * /etc/php56.d/php-fpm.conf
  * /etc/php56.d/fpm.d/*.conf

설정 파일 중, ___*.ini___ 파일들은 PHP language에 관련된 설정들이고, ___*.conf___ 파일들은 ***php-fpm*** daemon에 대한 설정 입니다.

여기서는 ***php-fpm*** daemon 설정에 대해서 기술 합니다. php langunage측면 설정이나 특이점은 위에서 기술한 사항을 참조 하시면 됩니다.

안녕 리눅스의 ***php-fpm***을 기본 상태로 구동을 하면 다음의 상태로 process가 뜨게 됩니다.


1. pid file
  * php7 : /var/run/php-fpm.pid
  * php56 : /var/run/php56-fpm.pid
2. error log
  * php7 : /var/log/fpm/php-fpm.log
  * php56: /var/log/fpm/php56-fpm.log
3. process owner : nobody.nbody
4. listen
  * php7 : /var/run/php-fpm-default.sock
  * php56 : /var/run/php56-fpm-default.sock
5. listen backlog : 1024
6. security.limit_extensions: .php

매우 busy한 site가 아니라면 이 기본 설정만으로도 운영을 하는데 크게 문제가 없습니다. 신경을 써 줘야 할 정도는 ***"security.limit_extension"*** 의 경우 application 별로 다른 확장자를 php로 구동해야할 경우가 있기 때문에 수정이 필요할 듯 판단 됩니다.

그 외에는 성능에 관련된 부분인 자세한 설정에 관해서는 [PHP-FPM 설정 문서](http://kr.php.net/manual/en/install.fpm.configuration.php)를 참고 하십시오.

또한, site를 여러개를 운영할 경우 site별로 pool을 만들어서 resource를 배분할 수도 있습니다.

### 7.2. php-fpm 설정

일단 기본적인 구동 방법을 설명 합니다. 자세한 사항은 설정 파일의 주석으로 보고 처리 하십시오.

여기서는, 기본 설정 파일은 건드리지 않고, ***/etc/php.d/fpm.d*** 에 설정을 overwrite 하거나 추가 하는 방향으로 운영의 미를 살립니다. :-) 이렇게 하면, 나중에 기본 설정에서 내가 무얼 변경했는지 확인하기가 쉽습니다.

아래의 내용으로 ***/etc/php.d/fpm.d/local.conf*** 를 생성 합니다.

```bash
[root@an3 fpm.d] cat local.conf
user = nobody
group = nobody

# IPv4 보다는 unix domain socket을 이용하는 것이 좀 더 성능이 좋습니다.
# local에서 웹서버와  연동을 할 것이라면 unix domain socket을 권장 합니다.
# 단, lighttpd와 연동을 할 경우, lighttpd가 unix domain socket과 연동이
# 되지 않으므로 IPv4 로 설정 하십시오.
#listen = 127.0.0.1:9000
listen = /var/run/php-fpm-default.sock
listen.mode = 0666
listen.backlog = 4096

chdir = /

pm = dynamic

pm.max_children = 40
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 40

php_admin_value[open_basedir] = /home/httpd:/usr/share/php/pear:/var/lib/php
php_flag[allow_url_fopen] = Off
[root@an3 fpm.d]
```

위의 설정은 기본 값도 인 것들도 있고, 변경된 값도 있지만, 대충 PHP-FPM을 구동할 때 서비스에 영향을 미칠 수 있는 것들을 나열해 놓은 것입니다.

### 7.3. php-fpm 구동

  간단한 ***php-fpm*** control 방법에 대하여 기술 합니다. ***php56-fpm*** package는 php-fpm 대신 php56-fpm을 사용하시면 됩니다.
  
  ***php-fpm***과 ***php56-fpm*** 은 동시에 운영이 가능 합니다.

  * 부팅시 php-fpm 시작하도록 설정
  ```bash
  [root@an3 ~]$ service php-fpm enable
  [root@an3 ~]$ # 또는
  [root@an3 ~]$ ntsysv-systemd
  ```
  * 부팅시 php-fpm 시작 하지 않도록 설정
  ```bash
  [root@an3 ~]$ service php-fpm disable
  [root@an3 ~]$ # 또는
  [root@an3 ~]$ ntsysv-systemd
  ```
  * php-fpm 시작
  ```bash
  [root@an3 ~]$ service php-fpm start
  ```
  * php-fpm 정지
  ```bash
  [root@an3 ~]$ service php-fpm stop
  ```
  * php-fpm 재시작
  ```bash
  [root@an3 ~]$ service php-fpm restart
  ```
  * php-fpm 상태 보기
  ```bash
  [root@an3 ~]$ service php-fpm status
  ```

## 8. Web server 연동

웹서버 연동은 php-fpm을 먼저 구동한 후에 하도록 합니다.

### 8.1 Apache

***/etc/httpd/conf.d/LoadModules.conf*** 에서 ***proxy_module*** 과 ***proxy_fcgi_module*** 의 주석을 해제 합니다.

```apache
#
# Proxy Modules
#
LoadModule  proxy_module            modules/mod_proxy.so
#LoadModule lbmethod_bybusyness_module  modules/mod_lbmethod_bybusyness.so
#LoadModule lbmethod_byrequests_module  modules/mod_lbmethod_byrequests.so
#LoadModule lbmethod_bytraffic_module   modules/mod_lbmethod_bytraffic.so
#LoadModule lbmethod_heartbeat_module   modules/mod_lbmethod_heartbeat.so
#LoadModule proxy_ajp_module        modules/mod_proxy_ajp.so
#LoadModule proxy_balancer_module   modules/mod_proxy_balancer.so
#LoadModule proxy_connect_module    modules/mod_proxy_connect.so
#LoadModule proxy_express_module    modules/mod_proxy_express.so
LoadModule  proxy_fcgi_module       modules/mod_proxy_fcgi.so
#LoadModule proxy_fdpass_module     modules/mod_proxy_fdpass.so
#LoadModule proxy_ftp_module        modules/mod_proxy_ftp.so
#LoadModule proxy_http_module       modules/mod_proxy_http.so
#LoadModule proxy_scgi_module       modules/mod_proxy_scgi.so
```

PHP-FPM 구동에 대한 기본 설정은 ***/etc/httpd/conf.d/php.conf*** 에서 ***php***와 ***php3*** 확장자에 대해서 PHP 동작을 하도록 설정이 되어 있습니다. FPM 설정에서 ***listen***을  IPv4로 했다면, 아래의 설정을 unix socket에서 IPv4 로 변경해 주십시오.

```apache

# for fastcgi configuration
#
<IfModule proxy_fcgi_module>
    <FilesMatch "\.(php|php3)$">
        # use IPv4
        #SetHandler  "proxy:fcgi://localhost:9000"
        
        # use unix domain socket
        SetHandler "proxy:unix:/var/run/php-fpm-default.sock|fcgi://localhost/"
    </FilesMatch>
</IfModule>
```

다음 httpd를 재시작 해 줍니다. 

```bash
[root@an3 ~]$ service httpd restart
```

위의 과정을 간단하게 정리를 하면 다음과 같습니다. ***php56-fpm*** 을 사용하면 설정 파일의 경로가 ***/etc/php56.d/fpm.d*** 입니다.

```bash
[root@host ~]$ yum install httpd php-fpm
[root@host ~]$ echo "listen = /var/run/php-fpm-default.sock" >> /etc/php.d/fpm.d/local.conf
[root@host ~]$ echo "listen.mode = 0666" >> /etc/php.d/fpm.d/local.conf
[root@host ~]$ service php-fpm start
[root@host ~]$ vi /etc/httpd/LoadModules.conf
...
LoadModule  proxy_module            modules/mod_proxy.so
LoadModule  proxy_fcgi_module       modules/mod_proxy_fcgi.so
...
[root@host ~]$ service httpd restart
```

### 8.2 lighttpd 1.4

lighttpd는 fastcgi 연동을 unix domain socket으로 할 수 없기 때문에, php-fpm의 listen 설정을 IPv4로 하십시오.

```bash
server.modules += ( "mod_fastcgi" )

fastcgi.server = (
    ".php" => (
        "localhost" => (
            "host" => "127.0.0.1",
            "port" => 9000,
            "broken-scriptfilename" => "enable",
            "allow-x-send-file" => "enable"
        )
    )
)
```

```bash
[root@an3 ~]$ service lighttpd restart
```

### 8.3 nginx

```nginx
server {
    listen       80;
    ...

    #
    # Fastcgi localhost
    #
    location ~ \.php($|/) {
        fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;

        if ( !-f $document_root$fastcgi_script_name ) {
            return 404;
        }

        include  params/fastcgi_params;
        #fastcgi_pass 127.0.0.1:9000;
        fastcgi_pass unix:/var/run/php-fpm-default.sock;

        fastcgi_index index.php;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        fastcgi_read_timeout 60;
    }
}
```

```bash
[root@an3 ~]$ service nginx restart
```

## 9. Composer 사용

안녕 리눅스에서 제공하는 PHP package 환경에 composer를 사용할 경우, php56-5.6.31-1, php71-7.1.9-1, php-7.0.23-1 버전 보다 낮은 버전이 설치 되어 있으면 안녕 리눅스의 PHP의 PHP_VERSION 에 "***AnNyung***" 이라는 문자열이 포함되어 있어 다음과 같이 "***Invalid version string***" 에러가 발생 합니다.

```bash
[root@an3 ~]$ ./composer.phar require monolog/monolog

  [UnexpectedValueException]
  Invalid version string "7.0.7AnNyung"

[root@an3 ~]$
```

이 문제를 해결하기 위하여 composer를 설치 한 다음, http://mirror.oops.org/pub/AnNyung/etc/composer 에서 이 문제가 수정된 최신 버전의 파일을 받아서 composer.phar을 바꿔치지 해 주거나, 또는, yum 을 이용하여 php를 최신 상태로 업데이트 해 주시면 해결이 됩니다.

직접 파일을 수정 하기 위해서는 다음의 작업 과정을 거칠 수 있습니다.



### 9.1 Composer 문제 해결

#### 9.1.1 composer 압축 해제

```bash
  [root@an3 ~]$ phar extract -f composer.phar composer/
```

#### 9.1.2 nomalize 함수 수정

*composer/vendor/composer/semver/src/VersionParser.php* 에서 ***nomalize*** 함수를 아래 패치 파일을 참조 하여 수정 합니다.
  
```patch  
diff -urNp composer.org/vendor/composer/semver/src/VersionParser.php composer/vendor/composer/semver/src/VersionParser.php
--- composer.org/vendor/composer/semver/src/VersionParser.php   2016-08-08 01:06:55.439483174 +0900
+++ composer/vendor/composer/semver/src/VersionParser.php   2016-08-08 01:10:54.268678206 +0900
@@ -105,6 +105,8 @@ if (null === $fullVersion) {
 $fullVersion = $version;
 }

+$version = preg_replace ('/AnNyung/i', '', $version);
+$fullVersion = preg_replace ('/AnNyung/i', '', $fullVersion);

  if (preg_match('{^([^,\s]++) ++as ++([^,\s]++)$}', $version, $match)) {
 $version = $match[1];
```

#### 9.1.3 phar package stub 파일 생성

stub 파일은 원본 composer.phar 에서 얻을 수 있습니다.

```bash
[root@an3 ~]$ phar stub-get -f composer.phar
#!/usr/bin/env php
<?php
/*
 * This file is part of Composer.
 *
 * (c) Nils Adermann <naderman@naderman.de>
 *     Jordi Boggiano <j.boggiano@seld.be>
 *
 * For the full copyright and license information, please view
 * the license that is located at the bottom of this file.
 */

// Avoid APC causing random fatal errors per https://github.com/composer/composer/issues/264
if (extension_loaded('apc') && ini_get('apc.enable_cli') && ini_get('apc.cache_by_default')) {
    if (version_compare(phpversion('apc'), '3.0.12', '>=')) {
        ini_set('apc.cache_by_default', 0);
    } else {
        fwrite(STDERR, 'Warning: APC <= 3.0.12 may cause fatal errors when running composer commands.'.PHP_EOL);
        fwrite(STDERR, 'Update APC, or set apc.enable_cli or apc.cache_by_default to 0 in your php.ini.'.PHP_EOL);
    }
}

Phar::mapPhar('composer.phar');
require 'phar://composer.phar/bin/composer';

__HALT_COMPILER(); ?>
[root@an3 ~]$
```

아쉽게도 redirection으로 파일을 직접 저장하면 좋겠지만, 되지를 않습니다. 출력된 결과물을 stub.php 라는 파일에 copy & paste 하도록 합니다.

#### 9.1.4 composer.phar 재 패키징

```bash
[root@an3 ~]$ phar pack -f composer-new.phar -s stub.php composer
[root@an3 ~]$
```