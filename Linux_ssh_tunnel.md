# Linux SSH tunnel 设置及使用

## 基本概念

- 隧道（tunneling）: 相当于一根虚拟网线，这根虚拟网线通过ssh加密实现

- 端口转发：在网络应用中，一个网口代码一个应用程序或应用服务，端口转发的意思是将远程应用服务端口从隧道中引入到本地中映射端口，或者将本地服务端口引出到远程主机中的映射端口

- 本地转发(导入应用)：将远程应用程序(端口1)引入到本地主机的映射端口2，本地实现将端口2转换成为端口1，所以其它内部不能连接远程端口1的客户可以通过连接本地的端口2实现访问。

- 远端转发(导出应用)：将本地应用程序(端口1)引出到远程主机的映射端口2，远端主机实现将端口2转换成端口1，所以其它远端不能连接本地内部端口1的客户可以通过连接远端口2实现访问。


## 原理

### 本地转发(解决本地其它主机不能访问外网主机的问题，将外网应用映射到本地端口)

当本地机器不能直接访问RemoteOtherIP:RemoteOtherPort时，通过LocalHost <=> sshServer建立ssh tunnel间接访问：

**本地其它LocalOther** => **LocalHost:LocalNewListenPort** => 本地转发 => RemoteSshServer) => **RemoteOtherIP:RemoteOtherPort**

```
# 在LocalHost上执行，本地作为中转机，本机也为服务接入机（即转发端口侦听机）
ssh -g -N -f -L LocalNewListenPort:通过sshServer能访问的RemoteIP:通过RemoteSshServer能访问的RemotePort user@RemoteSshSever
```

### 远端转发(解决外网其它主机不能访问私网主机的问题，将内同应用映付到公网端口)

当远端机器不能直接访问LocalOtherIP:LocalOtherPort时，通过RemoteSshServer <=> LocalHost建立ssh tunnel间接访问：

**远程其它RemoteOther** => **RemoteSshSever:RemoteNewListenPort** => RemoteSshServer => 本地转发 => **LocalOtherIP:LocalOtherPort**

```
# 在LocalHost上执行，本机作为中转机，远程机作为服务接入机（即转发端口侦听机）
ssh -g -N -f -R RemoteNewListenPort:通过LocalHost能访问的LocalOtherIP:通过LocalHost能访问的LocalOtherPort user@RemoteSshSever
# 使用ssh登录到RemoteServer上查看RemoteNewListenPort状态
netstat -tunlp|grep $RemoteNewListenPort && lsof -i :$RemoteNewListenPort
# 实际示例：将本机ssh服务分享到外网，然后可以通过命令：ssh -p 10022 localUser@20.236.71.98 访问本机sshq服务
ssh -g -N -f -R 10022:localhost:22 root@20.236.71.98
```

注意：要允许RemoteNewListenPort绑定在非环回地址上，需要在RemoteSshSever的sshd配置文件中启用"GatewayPorts"选项

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
- [SSH隧道：端口转发功能详解](https://www.cnblogs.com/f-ck-need-u/p/10482832.html)
