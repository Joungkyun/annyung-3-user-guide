# vim

### Description:
A version of the VIM editor which includes recent enhancements

vim package는 존재하지 않으며, 기본 패키지는 **vim-enhanced** 이다.

### Changes on AnNyung:
1. comment 색상 변경
2. _C / php / named / apache_ syntax 추가
3. **virc** import 에러 수정
4. **EDITOR** 환경 변수를 /usr/bin/vim 으로 지정 (_/etc/profile.d/vim.sh_)
5. _C / php / perl_ 파일에서 folder 기능 향상
 * 매핑키를 **"Z"**에서 **'+'**로 변경
 * function 선언 라인에서 **'+'**를 누르면 function 영역을 folding 함
 * 이 기능의 사용을 원하지 않으면, vimrc 또는 .vimrc에 다음 설정 셋팅
 ```ini
 let g:annyungfolding = 0
 ```
6. phpDocument 스타일의 phpfolding 추가
 * http://www.vim.org/scripts/script.php?script_id=1623 참조
 * 단축키 설정: 다음 설정을 vimrc 또는 .vimrc 에 셋팅
 ```ini
 map <F5> <Esc>:EnableFastPHPFolds<Cr>
 map <F6> <Esc>:EnablePHPFolds<Cr>
 map <F7> <Esc>:DisablePHPFolds<Cr>
 ```
7. native PHP manual 추가 (_Shift + k_)
8. checksyntax 플러그인 추가
 * 이 기능의 사용을 원치 않으면, vimrc 또는 .vimrc에 다음 설정 셋팅
 ```ini
 let g:checksyntax_auto = 0
 ```

### Sub packages:
* **vim-common** - The common files needed by any version of the VIM editor
* **vim-enhanced** - A version of the VIM editor which includes recent enhancements
* **vim-filesystem** - VIM filesystem layout
* **vim-minimal** - A minimal version of the VIM editor