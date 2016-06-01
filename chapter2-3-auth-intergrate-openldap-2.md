# OpenLDAP SSL 설정

> ***목차***
> 1. 인증서 준비
> 2. LDAP 설정
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








여기서는 [StartCOM](http://startssl.com) 의 Class2 인증서를 가지고 설정 하는 예를 듭니다. Self Sign 인증서를 생성하는 사용하는 방법은 [http://www.server-world.info/en/note?os=CentOS_7&p=openldap&f=4](http://www.server-world.info/en/note?os=CentOS_7&p=openldap&f=4)를 참고 하십시오.

StartCOM의 인증서를 예로 드는 이유는, 일단 공인 인증서 이면서 가격이 가장 싸기 때문입니다. 59$로 2년짜리 인증서를 발급 받을 수 있으며, \*.domain.com 과 같은 astrik 인증서를 생성할 수 있으며, 도메인도 여러개를 추가할 수 있기 때문 입니다. 1년 짜리 class 1 인증서는 무상으로 계정당 5개 까지 만들 수 있습니다. [***startssl 인증서***](https://www.google.co.kr/#q=startssl+%EC%9D%B8%EC%A6%9D%EC%84%9C)로 구글 검색을 하시면 한글로 된 발급을 위한 많은 글들을 보실 수 있습니다.

다른 공인 인증서도 크게 다르지는 않으니 응용해 보십시오. Chain 인증서가 없는 경우에는 CA 인증서(PEM 방식)으로 등록 하시면 됩니다.

CA chain 인증서는 [https://startssl.com/root](https://startssl.com/root)에서 받을 수 있으며, PEM 방식으로 받으셔야 하며, 인증서 타입을 잘 살펴 보셔야 합니다. 필자는 발급 받은지가 꽤 되어서 ***Deprecated Intermediate CA Certificates*** 섹션의 [***StartCom Class 2 Primary Intermediate Server CA(pem)(SHA-2)***](https://startssl.com/certs/class2/sha2/pem/sub.class2.server.sha2.ca.crt)을 사용합니다.

server key는 암호를 제거해 주셔야 합니다. 암호 제거는 다음과 같이 할 수 있습니다.

```bash
[root@an3 ~]$ openssl rsa -in oops.org.key -out oops.org.decrypt.key
Enter pass phrase for oops.org.key: [비밀번호 입력]
writing RSA key
[root@an3 ~]$
```

위와 같이 실행을 하면, oops.org.decrypt.key 로 암호가 제거된 key file이 생성이 됩니다.

발급받은 인증서를 /etc/openldap/cert/pki 라는 디렉토리를 생성하고 복사를 합니다. 다음 소유권과 권한을 다음과 같이 설정을 합니다.

```bash
[root@an3 ~]$ chown ldap.ldap /etc/openldap/cert/pki/*.*
[root@an3 ~]$ chmod 600 /etc/openldap/cert/pki/*.key
[root@an3 ~]$ ls -al /etc/openldap/cert/pki
-rw-r--r-- 1 ldap ldap 2451 2015-01-29 02:58 oops.org.crt
-rw------- 1 ldap ldap 1679 2015-01-28 18:03 oops.org.decrypt.key
-rw-r--r-- 1 ldap ldap 2124 2016-03-31 17:33 startssl-sub.class2.server.ca.sha2.pem
```

##2. LDAP 설정

다음과 같이 ssl-conf.ldif 파일을 생성하여, ***ldapmodify*** 명령을 이용하여 ***OLC***에 반영 합니다.

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

##3. OpenLDAP 재시작

***OLC***를 이용하면 굳이 재시작 할 필요가 없지만, ***slapd*** 시작시에 기본으로 ***ldaps*** 프로토콜을 사용하지 않도록 실행이 되었기 때문에, 이를 수정해서 재시작 합니다.

***/etc/sysconfig/ldap*** 에서 ***SLAPD_LDAPS*** 의 값을 ***yes***로 수정 합니다.

```bash
[root@an3 ~]$ perl -pi -e 's/^SLAPD_LDAPS=.*/SLAPD_LDAPS=yes/g' /etc/sysconfig/ldap
[root@an3 ~]$ cat /etc/sysconfig/ldap | grep "^SLAPD_LDAPS="
SLAPD_LDAPS=yes
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

먼저, ldap client에서 인증서를 사용하기 위하여 ***/etc/openldap/ldap.conf***에 다음의 설정을 합니다.

```bash
[root@an3 ~]$ cat >> /etc/openldap/ldap.conf << EOF

TLS_CACERTDIR   /etc/openldap/certs
TLS_CACERT      /etc/openldap/certs/pki/startssl-sub.class2.server.ca.sha2.pem
EOF
[root@an3 ~]$
```

LDAP 매니저 권한으로 ***ldaps*** 프로토콜을 이용하여 로그인 테스트를 합니다. 어떤 권한으로 하여도 상관은 없습니다.

```bash
[root@an3 ~]$ ldapsearch -H ldaps://localhost -D "cn=manager,dc=oops,dc=org" -W
Enter LDAP Password:
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