# mariadb-aes256

### Description:

MariaDB에서 AES 128/192/256 encryption을 지원하는 UDF(User defined function) 제공

### Features:

* AES 128 encrypt and decrypt
  * key length : 16byte
  * If the length of the key is 16byte, AES256_ENCRYPT will operate in the same way as AES_ENCRYPT.
```bash
mysql> select HEX(AES256_ENCRYPT('strings', '0123456789abcdef'));
mysql> select AES256_DECRYPT(UNHEX('encrypted_hash_string'), '0123456789abcdef');
```
* AES 192 encrypt and decrypt
  * key length : 24byte
```bash
mysql> select HEX(AES256_ENCRYPT('strings', '0123456789abcdef01234567'));
mysql> select AES256_DECRYPT(UNHEX('encrypted_hash_string'), '0123456789abcdef01234567');
```
* AES 256 encrypt and decrypt
  * key length : 32byte
```bash
mysql> select HEX(AES256_ENCRYPT('strings', '0123456789abcdef0123456789abcdef'));
mysql> select AES256_DECRYPT(UNHEX('encrypted_hash_string'), '0123456789abcdef0123456789abcdef');
```

### Reference:
* https://github.com/Joungkyun/lib_mysqludf_aes256

### Dependencies:
* [mariadb-server](pkg-base-mariadb.md)

### Sub Packages:


### Releated Packages:
