# PHP

> 목차
1. 개요
2. package 구성
3. 설정 파일
4. 안녕 리눅스 Epoch

##1. 개요
안녕 리눅스 3은 CentOS/RHEL과 달리 ***PHP 7***을 기본 제공 합니다. 또한 기존의 ***PHP 5***를 사용하는 환경의 호환을 위하여 ***php56*** package를 제공합니다.

안녕에서 제공하는 ***php56*** package에는 ***PHP 5.3*** 이후 <u>deprecated 되어진 기능들을 사용할 수 있는 PHP 5.3 compatible 기능</u>이 있습니다. 5.3 및 5.4 에서 동작하는 code들은 이 기능을 사용하여 코드 수정 없이 사용하실 수 있습니다. (몇가지 예외가 있으며, 예외에 대해서는 PHP 5.3 comapatible mode에서 기술 합니다.)

***php56*** package는 apache module(libphp5.so)를 지원하지 않습니다.

안녕 리눅스에서 제공하는 web server(apache/httpd, lighttpd, nginx)들은 대부분 ***fastcgi*** protocol을 이용하여 PHP와 연동을 합니다. 또한 apache 2.4의 경우, ***event*** MPM이 정식으로 포함이 되었으며, prefork + worker의 hybird 형식이므로, ***prefork***의 성능과 ***worker***의 안전성을 모두 개선하였으므로, ***event*** MPM 에 ***fastcgi***로 연동 하는 것을 권고 합니다. (안녕 리눅스 apache의 기본 MPM은 ***event*** 입니다.)

또한, PHP7과 PHP5를 동시에 운영하여야 한다면 ***fastscgi*** 연동을 선택할 수 밖에 없습니다.(라고 강요 합니다 ^^)

PHP 7 package는 mod_php package(php-7.0.x-x.an3.x86_64.rpm)을 제공을 하고는 있으나, 아마도 곧 decprecated 될 예정입니다. (빌드 시간이 너무 오래 걸립니다 T.T)

안녕 리눅스의 PHP package 구성에 대해서는 패키지 일람의 ***[php](pkg-base-php.md)***와 ***[php56](pkg-addon-php56.md)***를 참조 하십시오.

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

###1. exec_dir (PHP_INI_SYSTEM) 기능

이 기능은 php 5.4 이전의 safe_mode_exec_dir 기능을 safe_mode가 아닌 경우에도 사용할 수 있도록  그리고 기능을 확장해서 만들어진 기능입니다. 현재 PHP의 경우 5.4 부터 safe_mode가 없어지면서 이 기능도 같이 없어졌지만, 안녕 리눅스이 PHP에서는 exec_dir으로 이름을 수정하여 제공 합니다.

이 기능이 만들어진 이유는 2004년 경 phpBB highlight bug로 KLDP system이 shell injection을 당하여 서버 전체의 데이터가 삭제된 사건이 있었으며, 그 이후로 방치하기 쉬운 시스템의 보안을 어떻게 향상시킬수 있을까 고민하면서 만들어진 기능 입니다. (https://kldp.org/node/45576 참조)

즉, 이 기능이 만들어진 이유는 PHP의 shell injection 공격을 원천적으로 방지하기 위한 목적입니다. 대부분의 shell injectiondl 이제는 무지가 아니라 코딩 실수에 의하여 발생합니다. 이 패치는 이런 코딩 실수에 의한 injectioin hole을 원천적으로 방지해 줍니다.

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
      system ('find . -type -f -exec rm -f {} \;`);                  // 실제 코드
      system ('/var/lib/php/bin/find . -type -f -exec rm -f {} \;'); // PHP 내부에서 치환되어 수행되는 코드
    ```

###2. disable_functions 기본 적용

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
  
###3. allow_include_extension 기능 추가

  ***php-fpm***의 security.limit_extensions 와 동일한 기능입니다. PHP가 opcode compile 전에 여기에 지정된 확장자가 아니면 compile을 하지 않도록 합니다.
  
  이 기능은 image header에 php code를 숨겨 놓고, code를 injection해서 image header에 숨겨진 php code를 수행하는 것을 방지 할 수 있습니다.
  
  ```php
  <?php
  system ('/path/upload/attack-sample.jpg');
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

###4. file upload시 image header의 injection code 여부 검사 기능 추가

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
  * upload_image_check_whitelist
    * 이미지 헤더 문자열 white list

```php
<?php
switch ( $_FILES['userfile']['error'][0] ) {
    case UPLOAD_ERR_INI_SIZE :
        $errmsg = 'The uploaded file exceeds the upload_max_filesize directive in php.ini.';
        break;
    case UPLOAD_ERR_FORM_SIZE :
        $errmsg = 'The uploaded file exceeds the MAX_FILE_SIZE directive that was specified in the HTML form.';
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
    default :
        $errmsg = null;
}
?>
```

다음 사항은 ***[php56](pkg-addon-php56.md)*** package 에만 해당되는 사항입니다.