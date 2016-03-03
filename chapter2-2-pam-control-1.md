# login 가능한 account 제한

LDAP이나 AD, NIS등으로 인증 통합을 했을 경우, login이 필요없는 account가 대부분 입니다. 또는 서버마다 경우가 틀릴 수 있습니다.

이런 경우, pam_access 모듈을 이용하여 필요한 account만 login을 가능하도록 설정 하는 방법에 대하여 기술 합니다.

```bash
[root@an3 ~]$ cat /etc/pam.d/password-auth-ac
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_tally2.so deny=4 unlock_time=120
auth        required      pam_env.so
auth        sufficient    pam_unix.so nullok nodelay try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so

account     required      pam_access.so
account     required      pam_tally2.so
account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type= difok=1 minlen=8 minclass=3
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok remember=4
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
[root@an3 ~]$
```

위의 설정과 같이 ***account*** section의 가장 상단에 ***pam_access.so***를 required로 설정 합니다. pam_access 모듈의 option에 대해서는 [***"man pam_access"***](http://linux.die.net/man/8/pam_access)를 참고 하십시오.

다음, ***/etc/security/access.conf*** 에서 login 가능한 account, group 설정을 하도록 합니다.

```bash
[root@an3 ~]$ cat /etc/security/access.conf
+:root wheel:ALL
+:emer kss oops:ALL
+:kldp:LOCAL
-:ALL:ALL
[root@an3 ~]$
```

문법은 다음과 같습니다.

    CONDITION:USER or GROUP:ORIGIN

문법에 대한 자세한 예제는 ***/etc/security/access.conf***를 참고 하십시오.

 * CONDITION
   * ***+*** 또는 ***-***로 설정.
   * ***+***는 PERMIT
   * ***-***는 DENY를 의미
 * USER or GROUP
   * user 또는 group 을 기록
   * 공백 문자를 구분자로 복수 등록 가능
   * user와 gruop을 따로 구분하지 않기 때문에 동일한 user와 gruop간에 의도치 않은 결과 발생 가능하니 주의해야 함
   * netgroup의 경우에는 ***@***를 앞에 붙여줍니다. (@netgroup)
 * ORIGIN
   * 접근 시도 장소(?)
   * tty name, IPv4, IPv6, domain name, LOCAL, NONE, ALL 을 사용 가능

상단의 ***/etc/security/access.conf*** 의 의미는 다음과 같습니다.

1. root와 wheel group login permit
2. emer/kss/oops account or group login permit
3. kldp account or group은 LOCAL에서만 permit
4. 1~3 조건에 해당하지 않으면, 모두 deny