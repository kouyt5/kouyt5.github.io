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

## gcc 版本管理