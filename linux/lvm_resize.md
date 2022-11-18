## Linux LVM root partition (xfs) extend

```bash
lvextend -l +100%FREE /dev/fedora_fedora/root
xfs_growfs /dev/mapper/fedora_fedora-root
```

## (Fedora ZFS)[https://zfs.m-jay.cn/kai-shi-shang-shou/fedora]

```bash
dnf remove -y zfs-fuse
dnf install -y https://zfsonlinux.org/fedora/zfs-release$(rpm -E %dist).noarch.rpm
dnf install -y kernel-devel zfs
modprobe zfs
echo zfs > /etc/modules-load.d/zfs.conf
```
