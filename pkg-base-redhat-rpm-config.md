# redhat-rpm-config

### Description:
AnNyung(CentOS) specific rpm configuration files

### Changes on AnNyung:
1. check_rhel 에서 annyung이 체크되지 않는 문제 수정
2. rpm2cpio-extract 명령 추가 (다음 명령과 동일)

  ```bash
[root@an3 ~]$ rpm2cpio package.rpm | cpio -idmv
```

### Sub packages:
