---
title: 智慧医疗事业部技术分享-docker搭建lnmp环境
date: 2018-8-31 09:44:32
tags: docker
---


### 0x01. Docker是什么

> Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。 ---摘自百度知道。

> Docker 是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、 bare metal、OpenStack 集群和其他的基础应用平台。 -- 摘自 Docker 中文社区。

### 0x02. Docker 能干什么,为什么用 Docker

- 1、环境配置的难题

    软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？因此，用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，编译安装一套LNMP环境，计算机必须先安装各种库，各种依赖，可能还要配置环境变量。各个软件之间还需要协同工作。这已经不是易事，配置一台环境就如此麻烦，换一台机器，就要重来一次，旷日费时。然而我们经常会遇到的情况是多人协同开发。每个人的环境各不相同，本地环境与生产环境也不相同。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。
    
- 2、虚拟机

    虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点。

    - 资源占用多
    
    虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。
    
    - 冗余步骤多
    
    虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录。
    
    - 启动慢
    
    启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行。

- 3、Docker

    Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

    Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

    总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

- 4、总结一下 Docker 的用途

    （1）提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
    
    （2）提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
    
    （3）组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。
    
   时至今日，docker的生态已经非常繁荣，越来越多的软件都支持docker部署，现在在服务器中安装一个软件，我们可以无脑在dockerHub中搜索镜像，仅仅需要几个命令，就可以安装，运行，并且无需为兼容性而担忧。而且随着云计算，微服务，PAAS这些名词的火热，以docker为代表的虚拟化技术已经成为互联网领域不可或缺的技术栈，还是值得深入研究一下的。


### 0x03. 多说无益，下面我们从零搭建一套Docker下的LNMP环境

宿主机:ubuntu 16.04 server 

Docker版本：


#### 更改源信息
全新安装的ubuntu，默认软件下载是从国外，由于你懂的原因，访问龟速，偶尔被墙，我们首先换成国内的源。

```bash
cd /etc/apt 
# 源信息都配置在sources.list 文件中，我们先备份此文件
sudo cp sources.list sources.list.bak 
# 打开sources.list
vi sources.list
```
删除原有信息，将下面的源信息粘贴到sources.list中

```text
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
```
更新源
```bash
sudo apt-get update

命中:1 http://mirrors.aliyun.com/ubuntu xenial InRelease
命中:2 http://mirrors.aliyun.com/ubuntu xenial-updates InRelease        
命中:3 http://mirrors.aliyun.com/ubuntu xenial-backports InRelease      
命中:4 http://mirrors.aliyun.com/ubuntu xenial-security InRelease        
正在读取软件包列表... 完成 
```

测试一下，国内的源下载速度应该是比较快的
```bash
#安装一个vim编辑器，默认server版本没有预装
sudo apt-get install vim
```

#### 设置root登录
ubuntu 默认禁止root用户登录，由于我们是自己用来学习的虚拟机，不涉及到安全问题，用root登录方便很多。

```bash
#仅需要给root账户设置一个密码。
sudo passwd root 
#输入两次密码后，reboot 重启系统。
#这次使用root登录，密码填写设置过的root密码。
```
#### 配置ssh远程访问并且允许root账号登录

由于是在windows下通过虚拟机来演示docker的安装，直接在虚拟机上操作shell不太友好。所以先配置一下远程登录，用xShell工具登录操作服务器。

```bash
#先安装ssh服务端程序
apt-get install openssh-server
#修改配置文件，允许root用户远程登录
vim /etc/ssh/sshd_config
```
找到下面的配置
```text
# Authentication:
LoginGraceTime 120
PermitRootLogin prohibit-password
StrictModes yes
```
更改为：
```text
# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin yes
StrictModes yes
```
重启ssh
```bash
service ssh restart
```
之后就可以用xShell远程登录服务器了，还可以用xFTP操作文件。准备工作完成。

#### 安装Docker

```bash
#由于apt官方库里的docker版本可能比较旧，所以先卸载可能存在的旧版本：
apt-get remove docker docker-engine docker-ce docker.io

#更新源
apt-get update

#安装以下包以使apt可以通过HTTPS使用存储库（repository）：
apt-get install -y apt-transport-https ca-certificates curl software-properties-common

#添加Docker官方的GPG密钥：
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

使用下面的命令来设置stable存储库：
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

#再更新一下apt包索引：
apt-get update

安装最新版本的Docker CE：
apt-get install -y docker-ce
```

在生产系统上，可能会需要应该安装一个特定版本的Docker CE，而不是总是使用最新版本：
```bash
#列出可用的版本：
apt-cache madison docker-ce

#选择要安装的特定版本，第二列是版本字符串，第三列是存储库名称，它指示包来自哪个存储库，以及扩展它的稳定性级别。要安装一个特定的版本，将版本字符串附加到包名中，并通过等号(=)分隔它们：
apt-get install docker-ce=<VERSION>
```
至此。docker已经安装完毕。我们来验证一下，证明全栈helloWord工程师又学会了一门新的技术。

```bash
docker run hello-world

##输出的信息
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete 
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```
hello-world是官方提供的测试镜像。在运行该命令时，docker将会做如下操作：首先检测本地是否有一个叫hello-world的镜像文件，如果没有，则从docker-hub(类似与github的的管理仓库)上查找名为hello-world的镜像，若存在则下载到本地，若不存在，则返回找不到镜像的错误。拿到镜像之后使用它创造一个容器（container）,然后运行容器，得到结果，docker容器的特性是如果该容器运行的镜像中没有可执行的服务，则使用后会立刻关闭该容器。

#### 准备工作
终于来到了此次的重点，但是在安装之前还是有一些工作要做

   - 添加镜像加速
   
        国内访问 Docker Hub 有时会遇到困难，此时可以配置镜像加速器。国内很多云服务商都提供了加速器服务，我们这次用阿里云举例，注册用户并且申请加速器，会获得如 “https://t13n67zy.mirror.aliyuncs.com” 这样的地址。我们需要将其配置到Docker引擎。
        
        ```bash
        #编辑docker.service 文件
        vim  /etc/systemd/system/multi-user.target.wants/docker.service
        #找到 ExecStart= 这一行，在这行最后添加加速器地址 --registry-mirror=<加速器地址> 
        
        #加完以后看起来是这样的：ExecStart=/usr/bin/dockerd --registry-mirror=https://t13n67zy.mirror.aliyuncs.com
        ```
        重新加载配置
        ```bash
        #重新加载配置
        systemctl daemon-reload
        
        #重新启动Docker
        systemctl restart docker
        
        #设置Docker开机启动
        systemctl enable docker
       ```
        验证加速器是否有效
        ```bash
        #查看docker进程
        ps -ef  | grep dockerd
        #如果从结果中看到了配置的 --registry-mirror 参数说明配置成功。
        ```
    
- 安装docker-compose

    docker-compose 是一个软件，用来管理多个容器的docker，使他们能协同运行，只需要配置一个yaml文件，然后只需要一个简单的docker-compose up 命令，就可以运行所有的docker容器，我们先来安装。

    ```bash
    apt-get install docker-compose
    ```
#### 搭建环境
我们知道lnmp作为phper常用的开发环境，代表的是linux nginx mysql php 这个组合，我们可以把这样一套环境打包进一个镜像中，这个称之为fat container ，我们是不建议这么做的。我们倾向于将nginx mysql php 作为三个独立的容器运行，这样做的好处是：1，每个容器的体积都很小。相对独立运行便于修改，排错。2，快速/简单的发布 —— 因为镜像非常小，可以更快地下载镜像，也可以更快地发布到不同的机器。3，提高安全性 —— 更少的代码和程序在容器中意味着更小的攻击面。
















