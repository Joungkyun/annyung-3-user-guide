# Chapter 2. Access Control
## 2. Shell login Control (with PAM)
### 2. login account chroot

```bash
#!/bin/bash

chroot="/chroot"
create_dir="bin dev home lib64 lib proc sbin var/tmp var/log usr"
bind_dir="home var/log usr proc dev"
root_bind="bin sbin lib64 lib"
read_only="dev bin sbin lib64 lib usr"


if [ "${chroot}" = "" -o "${chroot}" = "/" ]; then
    echo "Invalid base chroot path \"${chroot}\"" 1> /dev/stderr
    exit 1
fi


mkdir -p ${chroot}

case "$1" in
    start)
        mount | grep "${chroot}/usr" >& /dev/null
        [ $? -eq 0 ] && echo "Aleady constructed chroot" && exit 1

        if [ ! -d "${chroot}/tmp" ]; then
            mkdir -p ${chroot}/tmp
            chmod 1777 ${chroot}/tmp
        fi

        for cdir in ${create_dir}
        do
            mkdir -p ${chroot}/${cdir} &> /dev/null
        done

        for cbind in ${bind_dir}
        do
            mount --bind /${cbind} ${chroot}/${cbind}
        done
        mount -o bind /dev/pts ${chroot}/dev/pts

        for rbind in ${root_bind}
        do
            mount -o bind /usr/${rbind} ${chroot}/${rbind}
        done

        rsync -av /etc/ ${chroot}/etc/ >& /dev/null
        rm -f ${chroot}/etc/shadow*

        ;;
    *)
        umount ${chroot}/dev/pts
        for cbind in ${root_bind} ${bind_dir}
        do
            umount ${chroot}/${cbind}
        done
        ;;
esac

exit 0
```