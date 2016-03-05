# Google OTP를 이용한 2 factor 인증

이 문서는 ***Google-Authenticator***를 이용하여 2-factor 인증을 구현하는 방법에 대해서 기술 합니다. ***Google-Authenticator***는 ***HMAC-Based One-time Password (HOTP)*** 방식과 ***Time-based One-time Password (TOTP)*** 방식을 지원하며, 여기서는 ***TOTP***를 이용하여 구성을 합니다.

2-factor 인증은, password 유출이나 brute force attack 대응에 훌륭한 방어 수단이 될 수 있습니다. 이를 위하여

현재 안녕 리눅스에서 google OTP를 이용한 인증은 ssh와 apache에만 국한이 됩니다. 여기서는 ssh login에 대해서만 다루고, apache 인증에 대해서는 [Chapter 3.5.1.7 Apache 2.4 Google Authentificator(Google OTP) 인증](chapter3-5-web-acl-apache.md)를 참고 하십시오.

> 목차
1. Installation
2. PAM 설정
3. openssh 설정
4. client 설정

##1. Installation

먼저 google OPT를 사용하기 위해서는 ***google-authenticator package***가 필요 합니다.

```bash
[root@an3 ~]$ yum -y install google-authenticator
  ** 상략 **

Running transaction
  Installing : 10:google-authenticator-1.01.20160217-1.an3.x86_64   1/1
  Verifying  : 10:google-authenticator-1.01.20160217-1.an3.x86_64   1/1

Installed:
  google-authenticator.x86_64 10:1.01.20160217-1.an3

Complete!
[root@an3 ~]
```

##2. PAM 설정

설치가 완료 되었으면, ***PAM*** 설정을 합니다.

```bash
[root@an3 ~]$ cat /etc/pam.d/sshd
#%PAM-1.0
auth       required     pam_google_authenticator.so nullok try_first_pass no_increment_hotp
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
-auth      optional     pam_reauthorize.so prepare
  ** 하략 **
[root@an3 ~]$
```

***auth*** 제일 상단에 ***required***로 ***pam_google_authenticator*** 를 등록 합니다.

지정된 ***pam_google_authenticator***의 옵션은 내용은 다음과 같습니다.

    * nullok            - ~/.ssh/google-authenticator 가 없으면 1-factor 인증 진행
                          이 옵션이 없으면 무조건 2-factor 인증을 진행 함
    * try_first_pass    - client에서 2-factor 인증을 하지 못할 경우 암호 입력시에 
                          "PasswordVeri_code" 형식으로 처리 가능 (확인 못함)
    * no_increment_htop - vefication code 확인 실패시에 count를 올리지 않음
    
다른 옵션들에 대해서는 [google-authenticator README](https://github.com/google/google-authenticator/tree/master/libpam)를 참고 하십시오.

여기서 ***nullok*** 옵션을 지정하면, account에 google-authenticator secret 파일이 있을 경우에만 2-factor 인증이 진행 됩니다. 없을 경우에는 기존의 1-factor 인증으로 진행이 되게 됩니다. 일단 ***nullok**** 옵션이 없으면 login을 하지 못하므로 지정하도록 합니다.


##3. SSH 설정

다음, openssh 서버 설정 파일(*/etc/ssh/sshd_config*)에서 Verication code를 사용할 수 있도록 설정해 줍니다.

1. PasswordAuthentication yes
2. ChallengeResponseAuthentication yes
3. UsePAM yes

안녕 리눅스의 경우 ***PasswordAuthentication***와 ***UsePAM***은 기본으로 yes값으로 지정이 되어 있으므로, ***ChallengeResponseAuthentication*** 값만 yes로 변경해 주면 됩니다.

```bash
[root@an3 ~]$ cat /etc/ssh/sshd_config
    ** 상략 **

  # Change to no to disable s/key passwords
  #ChallengeResponseAuthentication no
  ChallengeResponseAuthentication yes

    ** 하략 **
  [root@an3 ~]$
```

설정 파일 변경 후, sshd를 재시작 해 줍니다.

```bash
  [root@an3 ~]$ service sshd restart
```

##4. Client 설정

여기까지 서버 설정은 완료 되었으며, 다음은 client 설정에 대해 기술 합니다.

### 4.1 OPT program 설치

먼저, OTP program을 설치 합니다. 

  * iPhone이나 Android의 경우에는 App Store에서 ***Google OTP***를 설치 합니다.
  * Smart Phone 이 없거나 PC에서 사용하고 싶은 경우에는 [WinAuth](https://winauth.com/download/)를 이용합니다.

### 4.2 Secret file 생성

일단, 서버 설정이 완료 되었더라도, 각 계정에서 google-authenticator secret file을 생성하지 않으면 PAM 설정에서 nullok를 설정했기 때문에 기존의 1-factor 인증(그냥 ID/PW로만) 으로 login이 됩니다. secert file을 생성하기 위해서는 다음의 작업을 합니다.

이 작업을 하기 위해서는 ***~/.ssh*** 디렉토리가 있어야 하니, 없으면 먼저 생성을 해 주기 바랍니다. (퍼미션은 700으로)

```bash
  [root@an3 ~]$ su - bbuwoo
  [bbuwoo@an3 ~]$ mkdir ~/.ssh
  [bbuwoo@an3 ~]$ chmod 700 ~/.ssh
  [bbuwoo@an3 ~]$ google-authenticator -t -d --label=an3.oops.org --issuer=oops.org -r 3 -R 30
  https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/an3.oops.org%3Fsecret%3D65SANAUX4QX7OM5F%26issuer%3Doops.org

                  [QR-CODE-IAMGE]

  Your new secret key is: 65SANAUX4QX7OM5F
  Your verification code is 424981
  Your emergency scratch codes are:
    77133341
    84939993
    94216211
    47785086
    28719987

  Do you want me to update your "/home/bbuwoo/.ssh/google_authenticator" file (y/n) y

  By default, tokens are good for 30 seconds and in order to compensate for
  possible time-skew between the client and the server, we allow an extra
  token before and after the current time. If you experience problems with poor
  time synchronization, you can increase the window from its default
  size of 1:30min to about 4min. Do you want to do so (y/n) By default, tokens are good for 30 seconds and in order to compensate for
  possible time-skew between the client and the server, we allow an extra
  token before and after the current time. If you experience problems with poor
  time synchronization, you can increase the window from its default
  size of 1:30min to about 4min. Do you want to do so (y/n) y

  [bbuwoo@an3 ~]$ 
```

실행을 하면 위와 같이 진행이 되는데, 콘솔에서 image 출력이 가능하면 QR code가 출력이 됩니다. 만약 출력이 되지 않는다면, 제일 상단의 URL로 접근을 하면 QRcode를 확인할 수 있습니다. 물론 QRcode가 없을 경우, ***"Your new scret key is: ....."*** 부분에 있는 ***Secret Key***를 이용하여 등록을 하면 됩니다. Client 설정에 대해서는 "***Google Authenticator***"로 검색을 하시면 많이 나오니 자세한 사항은 인터넷 검색을 해 보시기 바랍니다. ***중요한것***은, 위의 결과에서 secret key를 ***~/.ssh/google-authenficator.info*** 라는 파일을 만들어(파일 이름은 중요하지 않습니다.) 남들이 보지 못하게(***chmod 600 filename***) 저장해 두시기 바랍니다.

  > ***다시 한번 강조***  
  위의 결과에서 secret key를 잘 보존하십시오!

```bash
  [root@an2 ~]$ ssh bbuwoo@an3test.oops.org
  AnNyung LInux 3 (Labas)
  Login an3test.oops.org on an 18:23 on Friday, 19 February 2016

  Warning!! Authorized users only.
  All activity may be monitored and reported

  Verification code: ******
  Password: ******************
  Last login: Fri Feb 19 18:09:09 2016 from 1.116.49.24
  [bbuwoo@an3test ~]$ 
```

### 4.3 참고

보너스로, 안녕 리눅스의 경우, Google authenticator 설정을 해 놓으면, passowrd expire 체크를 하지 않도록 패치가 되어 있습니다. 즉, 꽁수를 좀 부려 보자면 ***Google Authentifator 설정을 하지 않은 서버에서, 계정에 ~/.ssh/google-authenticator 파일이 존재할 경우, password expire를 피해갈 수 있습니다.