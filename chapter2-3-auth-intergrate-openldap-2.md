# OpenLDAP SSL 설정

> ***목차***
> 1. 인증서 준비
>   1. 무료 SSL 인증서 발급 기관
>   2. Self sign 인증서
> 2. LDAP 설정
>  1. ldap_ssl을 이용한 등록
>  2. 직접 등록
>  3. ssl 설정 제거
> 3. OpenLDAP 재시작
> 4. 설정 확인

<br><br>

## 1. 인증서 준비

SSL 인증서는 공인된 기관에서 발급하는 SSL인증서와 본인이 직접 발급하는 Self sign 인증서가 있습니다. 이 중 후자인 Self sign 인증서는 private 인증서라고 하여 공인된 인증서로 취급을 받지 못합니다만, ldap 구성을 하는데 있어서는 사용함에 크게 문제는 없습니다. 다만, LDAP gui tool을 사용할 경우, private 인증서라는 경고창이 뜰 수 있기 때문에(사용하는데는 지장이 없습니다.), 무료 SSL 인증서에 대한 정보를 같이 기술 합니다.

또한, 기존에 SSL 인증서를 이미 가지고 있다면, 해당 인증서를 사용할 수도 있습니다. 다만, 대부분의 LDAP GUI tool들이 ***SAN(Subject Alternative Name)***을 지원하지 않기 때문에, Astrik 도메인이나, SAN으로 여러 도메인을 관리하는 경우라면, 인증서를 hostname 과 동일하게 발급하는 것이 좋습니다.

###1. 무료 SSL 인증서 발급 기관

1. StartSSL https://startssl.com
1. WoSign https://buy.wosign.com/free/?lan=en
1. Let's encrypt https://letsencrypt.org/

무료 인증서 발급에 대해서는 구글 검색을 하여 발급을 받도록 합니다. 회원 가입과 여러가지 주의 사항들이 있으며, 여기서 일일이 설명을 하기에는 양이 많으므로, 구글 검색을 이용하십시오.

위의 3개의 인증서를 테스트 한 결과, 발급을 하기 위한 편리성 및 발급 시간은 ***WoSign***이 가장 인상적이었습니다. ***Let's encrypt***는 발급 및 유지가 너무 불편하여 권장하지 않습니다.

발급 받은 인증서를 ***/etc/openldap/certs/pki*** 디렉토리에 풀어 놓습니다. 그리고, 소유권과 퍼미션 설정을 하고, key 파일의 암호를 해제 합니다. 암호 해제는 다음의 명령으로 할 수 있습니다.

***WoSign*** 인증서의 경우 여러가지 형식의 인증서가 있는데, 이 중 Apache 인증서를 이용하시면 됩니다.

```shell
[root@an3 pki]$ openssl rsa -in an3.pkg.oops.org.key -out an3.pkg.oops.org.decrypt.key
Enter pass phrase for an3.pkg.oops.org.key: [비밀번호 입력]
writing RSA key
[root@an3 pki]$ chown ldap:ldap *
[root@an3 pki]$ chmod 600 *.key
```


###2. Self sign 인증서

***ldap-auth-utils***에서 제공하는 ***ldap_ssl*** 명령을 이용하며 간단하게 self sign 인증서를 생성할 수 있습니다. ***ldap_ssl*** 명령을 ***-c*** 옵션과 함께 실행을 하면 */etc/openldap/certs/pki*에 ***ldap.crt***와 (key 암호가 제거된)***ldap.key*** 파일이 생성이 됩니다.

```shell
[root@an3 ~]$ ldap_ssl -c
* 1. 개인키 생성

Generating RSA private key, 1024 bit long modulus
............++++++
...........................................................++++++
e is 65537 (0x10001)
writing RSA key
Success create /etc/openldap/certs/pki/ldap.key


* 2. 서버키 생성

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:KR
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:Seoul
Organization Name (eg, company) [Default Company Ltd]:OOPS.ORG
Organizational Unit Name (eg, section) []:Cert Team
Common Name (eg, your name or your server's hostname) []:an3.pkg.oops.org
Email Address []:admin@domain.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: => 그냥 엔터(설정 안함)
An optional company name []: => 그냥 엔터(설정 안함)
Success create /etc/openldap/certs/pki/ldap.csr


* 3. 자체 사인 인증서 생성

Signature ok
subject=/C=KR/L=Seoul/O=OOPS.ORG/OU=Cert Team/CN=an3.pkg.oops.org/emailAddress=admin@domain.com
Getting Private key
Success create /etc/openldap/certs/pki/ldap.crt
[root@an3 ~]$
```


##2. LDAP 설정

***ldap-auth-utils***에서 제공하는 ***ldap_ssl*** 명령을 이용하여 간단하게 등록할 수 있습니다.

인증서 경로는 절대 경로를 사용하는 것이 정신 건강에 도움이 됩니다.

###1. ldap_ssl을 이용한 등록
```shell
[root@an3 ~]$ ldap_ssl -h
사용법: ldap_ssl [OPTIONS]
옵션:
    -C PATH          CA 인증서 [기본값: /etc/pki/tls/certs/ca-bundle.crt]
    -a               이 서버에 SSL 설정 추가
    -c               개인키 생성 및 서버 인증서 셀프 사인
                     다른 옵션은 무시 됩니다.
    -r               이 서버의 SSL 설정 제거
    -p PATH          개인키
    -s PATH          서버 인증서
[root@an3 ]$ ldap_ssl -a -p /etc/openldap/certs/pki/ldap.key \
                    -s /etc/openldap/certs/pki/ldap.crt

Your informations:
    Ca Certificate     : /etc/pki/tls/certs/ca-bundle.crt
    Server Certificate : /etc/openldap/certs/pki/ldap.crt
    Server Key         : /etc/openldap/certs/pki/ldap.key

등록 성공slapd 데몬을 재시작 하십시오!
[root@an3 ~]$
```

slapd daemon을 재시작 하라는 메시지가 나오지만, ***OLC*** 설정 방식의 특성 상 재시작 하지 않아도 설정 변경이 반영이 됩니다. 다만 slapd daemon이 여전이 ldaps port를 binding 하고 있기 때문에 ldaps port binding을 하지 않게 하려면 slapd daemon을 재시작 시켜 주어야 합니다.

###2. 직접 등록

다음과 같이 ssl-conf.ldif 파일을 생성하여, ***ldapmodify*** 명령을 이용하여 ***OLC***에 반영 합니다. 다음 예제는 StartSSL의 Class 2 인증서를 사용할 경우 입니다.

```bash
[root@an3 ~]$ cat ssl-conf.ldif
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/pki/oops.org.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/pki/oops.org.decrypt.key
[root@an3 ~]$ ldapmodify -Y EXTERNAL -H ldapi:/// -f ssl-conf.ldif
[root@an3 ~]$ rm -f ssl-conf.ldif # ldif 파일은 굳이 보관할 필요 없습니다.
```

###3. SSL 설정 제거

***ldap_ssl** 명령을 ***"-r"*** 옵션과 함께 실행 합니다.

```shell
[root@an3 ~]$ ldap_ssl -r
삭제 성공 slapd 데몬을 재시작 하십시오!
[root@an3 ~]$
```

slapd daemon을 재시작 하라는 메시지가 나오지만, ***OLC*** 설정 방식의 특성 상 재시작 하지 않아도 설정 변경이 반영이 됩니다. 다만 slapd daemon이 여전이 ldaps port를 binding 하고 있기 때문에 ldaps port binding을 하지 않게 하려면 slapd daemon을 재시작 시켜 주어야 합니다.


##3. OpenLDAP 재시작

***OLC***를 이용하면 굳이 재시작 할 필요가 없지만, ***slapd*** 시작 시에 기본으로 ***ldaps*** 프로토콜을 사용하지 않도록 실행이 되었을 경우, 이를 수정해서 재시작 해야 합니다.

RHEL/CentOS 6 또는 AnNyung 2 에서는 ***/etc/sysconfig/ldap*** 에서 ***SLAPD_URLS*** 의 값을 ***yes***로 수정 합니다.

```bash
[root@an3 ~]$ perl -pi -e 's/^SLAPD_LDAPS=.*/SLAPD_LDAPS=yes/g' /etc/sysconfig/ldap
[root@an3 ~]$ cat /etc/sysconfig/ldap | grep "^SLAPD_LDAPS="
SLAPD_LDAPS=yes
[root@an3 ~]$
```

RHEL/CentOS 7 또는 AnNyung 3 에서는 ***/etc/sysconfig/slapd***의 ***SLAPD_URLS***에 ldaps:/// 값을 확인 하십시오.

```bash
[root@an3 ~]$ cat /etc/sysconfig/slapd | grep SLAPD_URLS
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
[root@an3 ~]$
```


설정을 완료 했으면, ***slapd***를 재시작 합니다.

```bash
[root@an3 ~]$ service slapd restart
slapd 를 정지 중:                                          [  OK  ]
slapd (을)를 시작 중:                                      [  OK  ]
[root@an3 ~]$
```

##4. 설정 확인

먼저, ldap client에서 인증서를 사용하기 위하여 ***/etc/openldap/ldap.conf***에 다음의 설정을 합니다. CA 인증서 경로는 ldap_ssl 실행 시에 지정한 경로로 지정 합니다.

```bash
[root@an3 ~]$ cat >> /etc/openldap/ldap.conf << EOF

TLS_REQCERT     allow
TLS_CACERTDIR   /etc/openldap/certs
TLS_CACERT      /etc/pki/tls/certs/ca-bundle.crt
#TLS_CACERT     /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
EOF
[root@an3 ~]$
```

LDAP 매니저 권한으로 ***ldaps*** 프로토콜을 이용하여 로그인 테스트를 합니다. 어떤 권한으로 하여도 상관은 없습니다. (단, -Y EXTERNAL 권한은 ldapi:// 프로토콜만 사용 가능 합니다.)

```bash
[root@an3 ~]$ ldapsearch -H ldaps:/// -D "cn=manager,dc=oops,dc=org" -W
Enter LDAP Password: [암호 입력]
# extended LDIF
#
# LDAPv3
# base <> (default) with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# search result
search: 2
result: 32 No such object

# numResponses: 1
[root@an3 ~]$
```