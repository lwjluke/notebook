
# Win10/WSL2.0打造Linux环境

## 1. WSL2 setup on Windows 11

- 以管理员权限打开Windows Powershell：
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
- 更新wsl2.0并安装Linux
```
wsl --update
wsl --set-version 2
wsl --list --online
wsl --install OracleLinux_8_7 # 安装并进入Linux shell
echo -e "[boot]\nsystemd=true" > /etc/wsl.conf
exit # 退出Linux shell
wsl -t OracleLinux_8_7
wsl # 重启Linux进入systemd模式
```

## 2. [编译升级WSL2 linux内核](https://nxdong.com/wsl-update-kernel/)

- 编译WSL2内核，然后重启计算机
```
su -; cd ~
git clone https://github.com/microsoft/WSL2-Linux-Kernel
cd WSL2-Linux-Kernel
sed -i 's/CONFIG_LOCALVERSION=.*/CONFIG_LOCALVERSION==""/' Microsoft/config-wsl
make -j 8 KCONFIG_CONFIG=Microsoft/config-wsl bzImage && make headers_install
version=$(grep -o  '[0-9.]*' include/generated/utsrelease.h)
rm -f /usr/src/kernels/*; ln -sf $(readlink -f .) /usr/src/kernels/$version
cp ./arch/x86/boot/bzImage /mnt/d/aws-linux/bzImage-$version
echo -e "[wsl2]\nkernel=d:\\\\aws-linux\\\\bzImage-$version\nswap=0" > /mnt/c/Users/simon/.wslconfig
```
- 编译Oracle Linux UEK内核，然后重启计算机

```
su -; cd ~
dnf config-manager --set-enabled ol8_codeready_builder
yum install rpm-build dwarves
dnf download --source kernel-uek # 下载src.rpm
rpm -ivh kernel-uek-5.15.0-106.131.4.el8uek.src.rpm
rpmbuild -bb ~/rpmbuild/SPECS/kernel-uek.spec # 解压并预编译内核
cd ~/rpmbuild/BUILD/kernel-5.15.0/linux-5.15.0-106.131.4.el8uek
cp ~/WSL2-Linux-Kernel/Microsoft/config-wsl .config # 使用WSL的配置编译UEK内核
make -j 8 bzImage && make headers_install
version=$(grep -o  '[0-9.]*' include/generated/utsrelease.h)
rm -f /usr/src/kernels/*; ln -sf $(readlink -f .) /usr/src/kernels/$version
cp ./arch/x86/boot/bzImage /mnt/d/aws-linux/bzImage-$version-uek
echo -e "[wsl2]\nkernel=d:\\\\aws-linux\\\\bzImage-$version-uek\nswap=0" > /mnt/c/Users/simon/.wslconfig
```
- Oracle Linux UEK内核集成ZFS，然后重启计算机

```
su -; cd ~
wget https://github.com/openzfs/zfs/releases/download/zfs-2.2.0/zfs-2.2.0.tar.gz
tar xvf zfs-2.2.0.tar.gz
cd zfs-2.2.0
./configure --enable-linux-builtin && make -j 8 && make install # 编译ZFS
./copy-builtin ~/rpmbuild/BUILD/kernel-5.15.0/linux-5.15.0-106.131.4.el8uek
cd ~/rpmbuild/BUILD/kernel-5.15.0/linux-5.15.0-106.131.4.el8uek
echo "CONFIG_ZFS=y" >> Microsoft/config-wsl
make -j 8 bzImage && make headers_install
version=$(grep -o  '[0-9.]*' include/generated/utsrelease.h)
rm -f /usr/src/kernels/*; ln -sf $(readlink -f .) /usr/src/kernels/$version
cp ./arch/x86/boot/bzImage /mnt/d/aws-linux/bzImage-$version-uek-zfs
echo -e "[wsl2]\nkernel=d:\\\\aws-linux\\\\bzImage-$version-uek-zfs\nswap=0" > /mnt/c/Users/simon/.wslconfig
```
- 验证WSL

```
uname -a # 显示WSL当前内核为：Linux Luke 5.15.0-106.131.4.el8uek.x86_64.debug
zpool status # 显示：no pools available
```

## 3. WSL2 支持物理磁盘

- 使用管理员权限打开Powershell，用以下命令列出物理磁盘

```
wmic diskdrive list brief # 显示：\\.\PHYSICALDRIVE0，\\.\PHYSICALDRIVE2
```

- 将物理盘1及物理盘2传入WSL2管理

```
wsl --mount \\.\PHYSICALDRIVE1 --bare # 以祼盘方式导入磁盘1
wsl --mount \\.\PHYSICALDRIVE2 --bare # 以祼盘方式导入磁盘2
wsl -u root zpool import zstore # 磁盘1及磁盘2组成ZPOOL zstore
wsl -u zpool status # 显示ZFS文件系统信息
wsl --unmount \\.\PHYSICALDRIVE1
wsl --unmount \\.\PHYSICALDRIVE2 # 使用完毕完后卸载磁盘
```

## 4. Upgrade Oracle Linux 8 to 9

```
grub2-mkconfig -o /boot/grub2/grub.cfg
leapp preupgrade --oraclelinux
leapp upgrade --oraclelinux
```


