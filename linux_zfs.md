# Linux ZFS

## 性能测试

```
yum install bonnie++
bonnie++ -d /mnt -r 8192 -s 16384 -u 0 | bon_csv2html | tee /root/xfs.html
```

创建加密ext4卷
```
zfs create -V 10g  -o encryption=on -o keyformat=passphrase  zfs/ext4_enc
mkfs.ext4 /dev/zvol/zfs/ext4_enc
mkdir -p /mnt/zvol_ext4_enc
mount /dev/zvol/zfs/ext4_enc /mnt/zvol_ext4_enc
cd /mnt/zvol_ext4_enc
bonnie++ -d /mnt/zvol_ext4_enc -r 8192 -s 16384 -u 0 | bon_csv2html | tee /root/zvol_ext4_enc.html
```