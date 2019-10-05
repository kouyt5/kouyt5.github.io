---
layout: post
title: ubuntu 错误解决方案
date: "2019-10-06 00:49:00 +0800"
tags: [ubuntu]
---

# ubuntu遇到的问题解决方案汇总
---

## 目录
* [apt-get update 404 error](#apt-get-update-404)

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
