---
title: windows SSH技巧
date: "2020-12-23 23:30:00 +0800"
tags: [工具]
---

# 废话
需要用到ssh的场景有很多，特别是最近兴起的 vscode 远程开发，使得ssh成为开发环境的重要一环。远程连接中，由于设置配置用户名和密码，因此安全是很重要的一环。ssh中默认是通过用户名和密码就可以登录。为了增强安全性，可以使用rsa公钥机制，更进一步还可以使用ip白名单机制防止不明用户ip的登录，防范黑客攻击。实际上，之前自己实验室的机器都是属于裸跑状态，但今天和朋友交流后觉得这样还是太不小心了，千万不要等到出现问题了才来悔恨。于是，准备将自己公网可以访问的服务器尽量设置成使用rsa加密方式ssh访问。

关于ip白名单机制的问题，这样可以更进一步加强安全性能，比如学校为了防止校外用户访问，设置了ip白名单，导致实验室的公网主页只允许校园网用户访问，虽然安全性提高了，但给我们造成了极大的困扰。显然我们有在校外进行访问的需求，学校给出的唯一的解决方式是通过 VPN 方式。然而学校提供的VPN太垃圾了，**占用内存**不说，还很**卡顿**，每次登录还有**过期时间**，也就意味着每次都要登录才能访问资源。好在有相当多的解决方案，可以完全避开使用VPN软件。例如**HTTP 代理**。因为大部分人日常用机都是 windows，而且 windows 是默认支持http代理的，也就是说，我这样配置后，就基本不管了，每次开机都会一直使用这个代理，只要我代理服务器架设在校园内，我的电脑从某种意义上说将一直在学校的网段内，岂不美哉。

于是乎，开干...

# http代理服务器搭建[refer](https://www.myfreax.com/how-to-install-and-configure-squid-proxy-on-ubuntu-18-04/)
比较好用的代理服务器是squid，docker搭建代码见 https://github.com/kouyt5/all-in-one/tree/master/lonely-app/squid
在`lonely-app/squid`下运行这个命令就在本地3128端口，开启代理服务器，使用windows或着火狐浏览器即可开启代理。
```
docker-compose up
```

windows上的配置例如：![代理.png](/assets/resource/代理/代理.png)
此时的代理还需要使用密码，密码配置如下

## 密码配置
linux命令行下输入以下命令
```
printf "USERNAME:$(openssl passwd -crypt PASSWORD)\n"
```
然后复制到 `lonely-app/squid/squid/htpasswd` 文件中即可，其中USERNAME和PASSWORD需要你指定你想设置的用户名和密码。
## v2ray
这个翻墙工具也可以用来做代理，具体配置详见我的[GitHub](https://github.com/kouyt5/all-in-one/tree/master/swarm/v2ray)
在这个目录下，使用如下命令，便可以在1080端口开启监听，不需要密码。
```
docker-compose -f docker-compose-cn.yaml up
```
# ssh 相关配置
ssh有一些参数在连接时候可以指定，有两种方式，一种是以命令的方式，另一种是配置文件的方式。为了方便仅仅记录配置文件的方式。在用户根目录的`.ssh/config` , 若没有请新建一个。如果没有其他设置，该文件的内容应该如下：
```
Host 211.83.110.70
  HostName 211.83.110.70
  Port 22
  User chenc
```
## ssh配置指定私钥
在配置文件中加入如下命令
```
IdentityFile /path/to/you/id_rsa
```
最后的文件内容为：
```
Host 211.83.110.70
  HostName 211.83.110.70
  Port 22
  User chenc
  IdentityFile /path/to/you/id_rsa
```
## SSH配置代理
在配置文件中加入如下命令
```
ProxyCommand D:\programs\connect.exe -H 211.83.110.70:1080 %h %p
```
其中connect.exe文件可以在git的安装文件夹中复制，一般是`C:\Program Files\Git\mingw64\bin`，注意路径最好是没有中文的。
## SSH防止断开
在本地配置文件中加入如下命令
```
ServerAliveInterval 60
```
网上说还可以在**服务端**加一行配置
```
ClientAliveInterval 60 # 这里的Client没错
```