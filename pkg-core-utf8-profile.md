# utf8-profile

### Description:

로컬 문자셋과 UTF-8간의 전환을 위할 프로파일

### Features:
1. /etc/sysconfig/utf8-profile 에 지정되어 있는 문자셋과 UTF-8간의 LANG 환경 변수 전환 alias 제공

### Reference:

* /etc/profile.d/utf8.sh
* /etc/sysconfig/utf8-profile

```bash
[root@an3 x86_64]$ alias u
alias u='shopt -s nocasematch; [[ "$LANG" =~ \.utf[-]?8$ ]] && [ "$OLD_LANG" != "$CONVERSION_LANG" ] && OLD_LANG="${CONVERSION_LANG}" ; [[ "$LANG" =~ \.utf[-]?8$ ]] && NEW_LANG="${OLD_LANG:=${CONVERSION_LANG}}" || NEW_LANG="${LANG%%.*}.UTF-8"; export OLD_LANG="$LANG" && export LANG="$NEW_LANG" && export USER_LANG="$NEW_LANG" && export LC_ALL="$NEW_LANG"; echo "Changed system charset to {${LANG}}"; shopt -u nocasematch'
[root@an3 x86_64]$ cat /etc/sysconfig/utf8-profile
# Locale conversion configuration
#
# /etc/profile.d/utf8.sh
#     conversion between UTF-8 and CONVERSION_LANG
#

CONVERSION_LANG="ko_KR.eucKR"
[root@an3 x86_64]$ u
Changed system charset to {ko_KR.eucKR}
[root@an3 x86_64]$ echo $LANG
ko_KR.eucKR
[root@an3 x86_64]$ u
Changed system charset to {ko_KR.UTF-8}
[root@an3 x86_64]$ echo $LANG
ko_KR.UTF-8
```

### Dependencies:
* None

### Sub Packages:
* None

### Releated Packages:
* None
