## Linux LVM root partition (xfs) extend

```bash
lvextend -l +100%FREE /dev/fedora_fedora/root
xfs_growfs /dev/mapper/fedora_fedora-root
```

