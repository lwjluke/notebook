# Linux SSH tunnel 设置及使用

## 基本概念

- 隧道（tunneling）: 相当于一根虚拟网线，这根虚拟网线通过ssh加密实现

- 端口转发：在网络应用中，一个网口代码一个应用程序或应用服务，端口转发的意思是将远程应用服务端口从隧道中引入到本地中映射端口，或者将本地服务端口引出到远程主机中的映射端口

- 本地转发(导入应用)：将远程应用程序(端口1)引入到本地主机的映射端口2，本地实现将端口2转换成为端口1，所以其它内部不能连接远程端口1的客户可以通过连接本地的端口2实现访问。

- 远端转发(导出应用)：将本地应用程序(端口1)引出到远程主机的映射端口2，远端主机实现将端口2转换成端口1，所以其它远端不能连接本地内部端口1的客户可以通过连接远端口2实现访问。


## 原理

## 实操

### Local本地转发(导入应用)

将远程服务导入可供其它内部访问，通过内部中转主机中转转发，从而其内部其它主机也能访问远程服务。

内部中转主机建立隧道并实现转发，通过建立端口2的侦听端口，将内部其它主机访问端口2的流量转换成远程主机的端口1。

其中的应用之一是科学上网。

通用命令：本机上的 forwardingPort 将会被监听，访问本机的 forwardingPort，就相当于访问 targetIP 的 targetPort，ssh隧道建立在本机与 sshServer 之间。

```
ssh -f -N -g -L LocalPort:targetIP:targetPort user@sshServerIP
telnet MyIP:LocalPort #=>sshServerIP=> targetIP:targetPort
```

实例：将43.23.1.13:8822转换成127.0.0.1:822，所以内部主机访问127.0.0.1:822即访问远程43.23.1.13:8822。

```
ssh -f -N -g -L 8822:127.0.0.1:822 root@43.23.1.13
```

### Remote远程转发(导出应用)

将本地服务导出可做其它远程访问，通过远程中转主机中转转发，从而其远程其它主机也能访问内部服务。

远程中转主机建立隧道并实现转发，通过建立端口2的侦听端口，将远程其它主机访问端口2的流量转换成内部主机的端口1。

其中的应用之一是将内网中的服务器提供给外网访问。

通用命令：sshServer 上的 forwardingPort 将会被监听，访问 sshServer 上的 forwardingPort，就相当于访问 targetIP 的 targetPort，ssh 隧道建立在本机与 sshServer 之间。

```
ssh -N -R serverPort:targetIP:targetPort user@sshServerIP
telnet sshServerIP:serverPort #=LocalHost=> targetIP:targetPort
```

实例：将10.216.29.19:22转换成43.23.1.13:922，所以远程主机访问43.23.1.13:922即访问内部10.216.29.19:22。

```
ssh -R 922:10.216.29.19:22 root@43.23.1.13
```

## 参考文档

- [SSH隧道简明教程](https://www.lixueduan.com/posts/linux/07-ssh-tunnel/)
