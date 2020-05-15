---
title: ubuntu 相关
date: "2019-10-06 00:49:00 +0800"
tags: [ubuntu]
---

# ubuntu
包含从ubuntu学到的linux相关知识与踩过的坑,不停更新中
## 目录
* [apt-get update 404 error](#apt-get-update-404)
* [系统环境文件](#系统环境文件)

## apt-get-update-404

在使用命令 `sudo apt-get update` 时，终端出现404错误，之前都直接忽略了，但是今天在安装Jekyll时发现报错：
```
+ rvm_error 'There has been an error while updating your system using `apt-get`.
It seems that there are some 404 Not Found errors for repositories listed in:

    /etc/apt/sources.list
    /etc/apt/sources.list.d/*.list
```
于是我使用 `apt-get update` 发现这样的错误：
```
 错误:13 http://ppa.launchpad.net/gias-kay-lee/npm/ubuntu bionic Release
  404  Not Found [IP: 91.189.95.83 80]
```
但是我访问这个网站又是可以的，这就有点怪了，好吧，我也不知道，直接 google 找到这篇文章 [无法安全地...](https://blog.csdn.net/chenbetter1996/article/details/80255552) 直接删掉 `/etc/apt/sources.list.d` 目录下对应错误ip地址的文件即可。

## 系统环境文件
原博客：[Linux如何修改环境变量PATH](https://blog.csdn.net/gui951753/article/details/79166236)
Linux系统环境变量配置文件：
+ `/etc/profile` : 在登录时,操作系统定制用户环境时使用的第一个文件 ,此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。
+ `/etc/environment` : 在登录时操作系统使用的第二个文件, 系统在读取你自己的profile前,设置环境文件的环境变量。
+ `~/.profile` :  在登录时用到的第三个文件 是.profile文件,每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。
+ `/etc/bashrc` : 为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.
+ `~/.bashrc` : 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。

### 系统启动
+ 在`/etc/init.d` 目录下，包含系统的各种命令，例如`/etc/init.d/networking stop`停止网络服务。
+ 在`/etc/rc.local/`文件中，包含系统启动时运行的脚本，也就是说把脚本写入这个文件系统启动的时候就会执行。
+ service调用`/etc/init.d/`和`/etc/init`下的脚本 eg:`service tomcat status`
+ `/etc/systemd/system`文件夹下放置开机自启动文件，例如tomcat配置文件。使用systemctl管理.`systemctl status tomcat`
## gcc 版本管理

### gcc 安装
添加ppa源：
```
$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
$ sudo apt-get update
$ sudo apt-get install gcc-5 g++-5
```
但是现在输入 `gcc --version` 发现还是以前的版本。因为linux系统有一个包管理工具，默认托管gcc的原因.输入下面命令，将gcc5添加到包管理工具中
```bash
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50
```
选择版本：
```bash
$ sudo update-alternatives --config gcc
```
有如下输出：
```
有 2 个候选项可用于替换 gcc (提供 /usr/bin/gcc)。

  选择       路径            优先级  状态
------------------------------------------------------------
  0            /usr/bin/gcc-4.8   100       自动模式
  1            /usr/bin/gcc-4.8   100       手动模式
* 2            /usr/bin/gcc-5     50        手动模式

要维持当前值[*]请按<回车键>，或者键入选择的编号：
```
选择你的版本即可。

## nvidia 驱动安装
### ppa安装
通过添加ppa源的方式，这种方式最简单，但是发现不能获取到最新的驱动。
```bash
sudo add-apt-repository ppa:graphics-drivers/ppa      //添加ppa库到系统中
sudo apt update         //  更新
 
sudo ubuntu-drivers devices // 显示可以安装的nvidia驱动
```

### 官方驱动安装
到官网找到自己的驱动 https://www.geforce.cn/drivers
下载到本地后赋予x权限
```bash
$ chmod a+x NVIDIA-Linux-x86_64-440.82.run
```
因为开着桌面图形界面，必须关掉才能安装驱动。首先查看桌面是否运行：
```bash
ps aux | grep X
```
发现如下输出：
```bash
root     26848 70.0  0.0 299724 52636 tty7     Ssl+ 01:07   0:00 /usr/lib/xorg/Xorg -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
```
通过 `sudo kill -9 26848` 是无法关掉进程的，它会自动重启，因此需要如下命令关掉 `lightdm` 进程:
```bash
sudo /etc/init.d/lightdm stop
```
现在就可以愉快的安装驱动了。
