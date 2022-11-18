## Upgrade Fedora 34 to verison 36

```bash
dnf upgrade
dnf --refresh upgrade
dnf install dnf-plugin-system-upgrade --best
dnf system-upgrade download --refresh --releasever=36
dnf system-upgrade reboot
yum update
```

Network configure:

```bash
sed -i 's@^#\?\(DNS=.*\)@DNS=10.83.50.140 8.8.8.8@g' /etc/systemd/resolved.conf
```

## Install xrdp:

```bash
yum group install "KDE Plasma Workspaces"
yum install -y xrdp
systemctl enable --now xrdp
# If Firewalld is running, allow RDP port.
firewall-cmd --add-port=3389/tcp --permanent
firewall-cmd --reload
```
xrdp support sound: [pulseaudio-module-xrdp](https://github.com/neutrinolabs/pulseaudio-module-xrdp/wiki/Build-on-Fedora)

```bash
dnf install mock
usermod -a -G mock $USER
# Log out and log in again
dnf download --source pulseaudio
mock ./pulseaudio-*.src.rpm

yum remove pipewire-pulseaudio
dnf install autoconf automake make libtool libtool-ltdl-devel pulseaudio-libs-devel git
git clone https://github.com/neutrinolabs/pulseaudio-module-xrdp.git
cd pulseaudio-module-xrdp
export PULSE_DIR=/var/lib/mock/fedora-36-x86_64/root/builddir/build/BUILD/pulseaudio-15.0
./bootstrap && ./configure PULSE_DIR=$PULSE_DIR &&　make && make install
```

## Install chrome browser:

```bash
# CentOS
cat << EOF > /etc/yum.repos.d/google-chrome.repo
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/x86_64
enabled=1
gpgcheck=1
gpgkey=https://dl.google.com/linux/linux_signing_key.pub
EOF

yum install google-chrome-stable

# Fedora
yum install fedora-workstation-repositories
yum config-manager --set-enabled google-chrome
yum install google-chrome-stable
```

## Install Microsoft Edge browser:

```bash
rpm --import https://packages.microsoft.com/keys/microsoft.asc
yum-config-manager --add-repo https://packages.microsoft.com/yumrepos/edge
yum install microsoft-edge-stable
```

## Install vscode:

```bash
cat << EOF > /etc/yum.repos.d/vscode.repo
[code]
name=Visual Studio Code
baseurl=https://packages.microsoft.com/yumrepos/vscode
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc
EOF

rpm --import https://packages.microsoft.com/keys/microsoft.asc
yum install code
```

## Install Chinese fonts:

```bash
yum install google-noto-sans-simplified-chinese-fonts
yum install google-noto-cjk-fonts google-noto-sans-cjk-fonts

yum install ibus-rime
git clone https://github.com/Openvingen/rime-zhengma.git
cp rime-zhengma/{zhengma,zmbig,zmb,zmjd,pinyin123}.*.yaml  /usr/share/rime-data
sed -i -e  '/schema_list:/a  - schema: zhengma' /usr/share/rime-data/default.yaml
```

## Install Podman & Docker container

```bash
yum-config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
systemctl enable --now docker

# podman
dnf install podman
```
