# genpasswd

### Description:

SHELL에서 md5, sha256, sha512 방식의 암호화 문자열을 생성할 수 있다.

### Features:

md5, sha256, sha512(기본) 방식의 hash 문자열을 생성할 수 있습니다.

```bash
[root@an3 ~]$ genpasswd -h
OC ERROR: Unsupported option -h

Usage: genpasswd [OPTION]
       -i      given password with STDIN
       -m      Crypt method: md5 or sha256 or sha512 [default: sha512]
       -s      user define salt [default: random string]
```

사용 법은 ***passwd*** 와 거의 동일합니다.

***-m*** 옵션으로 암호화 방식을 선택할 수 있습니다. ***-m*** 옵션이 주어지지 않았을 경우에는 ***sha512*** 로 암호화 됩니다.

```bash
[root@an3 ~]$ genpasswd -m md5
Password:
Retype Password:
$1$ABHjIPRv$4OXE8aJZu1TVMLI6mKRqb1
[root@an3 ~]$
```

입력된 암호는 3 class의 조합을 체크 하여, 취약한 암호를 예방합니다. 3 class의 조합은 대문자, 소문자, 숫자, 특수문자 중 3개의 조합으로 이루어져야 합니다. 또한, 9자 이상의 길이를 요구 합니다.

```bash
[root@an3 ~]$ genpasswd -m md5
Password:
BAD PASSWORD: The password contains less than 3 character classes
[root@an3 ~]$  genpasswd -m md5
Password:
BAD PASSWORD: The password is shorter than 8 characters
[root@an3 ~]$
```

STDIN 으로도 가능 합니다. STDIN으로 생성할 경우에는 암호화 강도를 ~~체크하지 않습니다~~(1.0.0 부터는 체크를 합니다).

```bash
[root@an3 ~]$ echo "asdf" | genpasswd -m md5 -i
$1$KSoP78Zr$teF7WRd2Azz0svaUXTkIZ0
[root@an3 ~]$
```



### Reference:
* https://github.com/Joungkyun/genpasswd

### Dependencies:
* ~~[libolibc](pkg-core-olibc.md)~~ : 1.0.0 부터 필요 없음
* crypt

### Sub Packages:
* None

### Related Packages:
* None