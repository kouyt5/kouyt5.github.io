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

## nvidia-docker 运行失败
最近搭建一个运行在`docker`的语音识别模型，因为需要 GPU 支持，所以需要安装`nvidia-docker` 根据教程，运行如下命令：
```bash
sudo apt-get install nvidia-docker2
sudo pkill -SIGHUP dockerd
```
使用官网注册的账号登录才能下载的镜像：
```bash
nvidia-docker run --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -it --rm nvcr.io/nvidia/tensorflow:19.05-py3
```
报错：
```
chenc@Aone-Family:~$ nvidia-docker run --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -it --rm nvcr.io/nvidia/tensorflow:19.05-py3
docker: Error response from daemon: Unknown runtime specified nvidia.
See 'docker run --help'.
```
来自守护进程的错误响应，未知的nvidia运行环境。这是什么？？试过了一些方法，如重启nvidia-docker进程均无效，最后好不容易找到了问题的原因。
+ [https://github.com/NVIDIA/nvidia-docker/issues/578](https://github.com/NVIDIA/nvidia-docker/issues/578)
+ -> [https://github.com/NVIDIA/nvidia-docker/wiki...](https://github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions#why-do-i-get-the-error-unknown-runtime-specified-nvidia)

原句是这样说的：
>Make sure the runtime was registered to dockerd. You also need to reload the configuration of the Docker daemon.

就是说，nvidia-docker 的运行环境没有注册到 docker，导致 docker 找不到 daemon.
根据链接：
+ [https://github.com/nvidia/nvidia-container-runtime#...](https://github.com/nvidia/nvidia-container-runtime#docker-engine-setup)

终于找到了问题的解决方案，原来是原本的安装描述忽略了一些细节，安装nvidia-docker 的方法应该使用github上 [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) 的安装方法。算了我那样安装了也将就着用吧。补救的方法为：
+ [https://github.com/nvidia/nvidia-container-runtime#...](https://github.com/nvidia/nvidia-container-runtime#docker-engine-setup)
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/override.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --host=fd:// --add-runtime=nvidia=/usr/bin/nvidia-container-runtime
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
# configuration
sudo tee /etc/docker/daemon.json <<EOF
{
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
EOF
sudo pkill -SIGHUP dockerd
```
这里有一句很显眼的话：
>Do not follow this section if you installed the nvidia-docker2 package, it already registers the runtime.

这就有点儿尴尬了。但是我都运行成功了，不知道里面的细节是什么，毕竟docker的东西我还没有吃透呢，先不管了，能用就行。运行命令：
```
chenc@Aone-Family:~$ nvidia-docker run --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -it --rm nvcr.io/nvidia/tensorflow:
19.05-py3
                                                                                                                                    
            
================
== TensorFlow ==
================

NVIDIA Release 19.05 (build 6390160)
TensorFlow Version 1.13.1

Container image Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
Copyright 2017-2019 The TensorFlow Authors.  All rights reserved.

Various files include modifications (c) NVIDIA CORPORATION.  All rights reserved.
NVIDIA modifications are covered by the license terms that apply to the underlying project or file.

NOTE: MOFED driver for multi-node communication was not detected.
      Multi-node communication performance may be reduced.
# nvidia-smi
root@1e6676f1d5b2:/workspace# nvidia-smi
Mon Feb 17 14:16:51 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 430.64       Driver Version: 430.64       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce RTX 2060    Off  | 00000000:01:00.0  On |                  N/A |
| 44%   66C    P2   151W / 160W |   3037MiB /  5931MiB |     79%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```
这样应该没问题了吧？