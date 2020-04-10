---
title: openseq2seq语音识别环境搭建
date: "2020-2-18 21:26:00 +0800"
tags: [docker,语音识别]
---
# docker 简介
`docker` 是一个使用`GO`语言实现的容器，所谓容器，跟虚拟机有点类似，操作系统上运行一个容器相当于运行一个全新的操作系统。但是这个容器并不是完整的操作系统，只有很少量必要的进程，因此`轻量化`是它的特点之一。容器隔离了宿主的进程，所以看起来容器中的进程无法直接影响宿主系统。那为什么要多次一举，直接在宿主上运行程序不行吗？
问题的关键就在于隔离，很多程序需要各种各样的环境，直接在宿主上会破坏原来的环境，但这个程序要这个环境，那个程序要那个环境，难不成写程序之间打一架？我们经常会遇到这样一种情况：
>A:你的程序在我电脑上怎么跑不起来？有BUG吧？
>B:(有BUG，这还能忍？)我XXOOB....。老子电脑上运行得好好的，你自己想办法...
>A:.....

对于python，有`conda`等环境管理工具，还能够缓减这个问题，实际上，并不能完全解决，因为python程序可能还得依赖其他的资源，例如GPU、C++库等。在这种情况下，docker便是一个比较好的解决方案。使用docker，可以在给A程序的时候同时给他一个docker镜像，A同学便能相当简单的构建一个与你环境一模一样的虚拟环境。这样再也不会出现以上对话中的情景了。

## [OpenSeq2Seq](https://github.com/NVIDIA/OpenSeq2Seq) 平台搭建
该平台是 NVIDIA 开源的一个`Seq2Seq`的平台，集成包括：
+ 语音识别
+ 翻译
+ 文本转语音
+ 语言模型
+ 图片标注

这几大类的 Seq2Seq 前沿算法。我主要是为了搭建语音识别模型 `Jasper` 该模型取得了非常好的识别效果，在`Librispeech` test-clean 数据集上取得了`2.85`的 WER ,发表在 2019 的 INTERSPEECH 上。比起`DeepSpeech2`的`5.33`提升还是比较明显。
搭建步骤： [https://nvidia.github.io/OpenSeq2Seq/html/installation.html](https://nvidia.github.io/OpenSeq2Seq/html/installation.html)
以下是详细介绍
### 1.注册 NGC(NVIDIA GPU Container) 账号
因为需要获取官方镜像，刚开始我直接`docker pull <image name>`发现一个 unauthority 的错误，折腾了好久，比如到docker官网注册账号，都还是不行，后面仔细看文档才发现，还有一个链接...。这个链接详细说了如何pull 镜像。首先到[NGC Cloud](https://ngc.nvidia.com/catalog/all)注册账号。注册成功后：
+ 安装[NGC CLI](https://ngc.nvidia.com/setup/installers/cli)
+ 获取密钥配置本地账号信息[api-key](https://ngc.nvidia.com/setup/api-key)

具体命令上诉两个地址都有。
### 2.镜像下载与容器运行
我踩了一个坑，官方给的教程中有一步是`nvidia-docker`安装，实际上这个命令已经被弃用了,详细见[nvidia-docker运行失败](#nvidia-docker运行失败)，
运行命令：
```bash
nvidia-docker run --name open -v /path/to/data:/path/to/mount/ --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 -it -d nvcr.io/nvidia/tensorflow:19.05-py3
# 如果按照最新的nvidia docker安装方式：
docker run --gpus all ....
```
创建一个名字叫做 open 的 container,同时挂载主机的目录到container 中。然后执行命令：
```bash
docker exec -it open bash
```
进入到容器中
### 3.git代码到本地
```bash
git clone https://github.com/NVIDIA/OpenSeq2Seq.git
```
然后就好了，环境搭建完成。只需要预处理数据集就可以，其他`tensorflow`之类的都不需要安装。数据预处理需要安装相关的依赖，如`tqdm`,`sox`。运行命令：
```bash
bash scripts/run_all_tests.sh
```
测试环境有没有问题，需要30分钟，可省略。
训练：
```bash
python run.py --config_file=example_configs/speech2text/jasper-Mini-for-Jetson.py --mode=train_eval --enable_logs --continue_learning
```
完成！
### nvidia-docker运行失败
最近搭建一个运行在`docker`的语音识别模型，因为需要 GPU 支持，所以需要安装`nvidia-docker` 根据教程(我晕，[教程](https://nvidia.github.io/OpenSeq2Seq/html/installation.html) 过时了，还是 nvidia 官网的，教程是18年的，现在nvidia-docker2 已经 **deprecated** 了。。。)运行如下命令：
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

终于找到了问题的解决方案，原来是原本的安装描述忽略了一些细节，安装nvidia-docker 应该使用github上 [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) 的安装方法。算了我那样安装了也将就着用吧。补救的方法为：
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
