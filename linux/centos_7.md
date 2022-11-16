## Install xrdp:

```bash

yum group install "KDE Plasma Workspaces"
yum install -y xrdp
systemctl enable --now xrdp
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
