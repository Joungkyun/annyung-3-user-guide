# sudo

## Description:

특정한 유저에게 제한된 root 사용을 허락

## Changes on AnNyung:

1. _**-f**_ 옵션값에 **"/"**가 없을 경우, 지정된 파일을 _/etc/sudoers.d_ 에서 검색하도록 수정
2. sudo 시 보존할 환경변수에 다음 환경 변수 추가 \(_/etc/sudoers_\)
   * _USER\_LANG_
   * _ULANG_
   * _SSH\_LANG_

## Sub packages:

* **sudo-devel** - Development files for sudo

