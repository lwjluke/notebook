# FreeBSD 14 Install

## KDE

- 安装KDE软件

pkg install --quiet --yes kde5 plasma5-sddm-kcm sddm xorg
sysrc dbus_enable="YES" && service dbus start
sysrc sddm_enable="YES" && service sddm start

- KDE不允许root直接登录，需要创建一个新的普通用户：

adduser 
