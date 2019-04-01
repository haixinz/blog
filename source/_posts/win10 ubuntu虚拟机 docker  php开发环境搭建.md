---
title: win10 -> ubuntu虚拟机->docker  php开发环境搭建
date: 2018-9-31 09:44:32
tags: docker
---

记录一下用windows 搭建linux的docker php 开发环境，实现在windows下开发，在docker中运行。坑还是很多的。

关于docker的安装以及linux的一些设置，参见前一篇文章 [智慧医疗事业部技术分享-docker搭建lnmp环境](https://haixinz.github.io/2018/08/31/%E6%99%BA%E6%85%A7%E5%8C%BB%E7%96%97%E4%BA%8B%E4%B8%9A%E9%83%A8%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB-docker%E6%90%AD%E5%BB%BAlnmp%E7%8E%AF%E5%A2%83/).

本人环境：
- 宿主机：windows 10 （1709）
- 虚拟机软件：WMware 15 player
- 虚拟机系统：ubuntu 16.04
- 容器: docker 18.09

#### 设置文件夹共享
1，选择 虚拟机->设置->选项->共享文件夹，选择windows下你想要共享的文件夹。这里一般选项目目录。

2，安装VMware Tools，可以帮助你在linux下自动挂载目录，不用每次开机重新挂载。
点击菜单项，选取虚拟机(M) --> 安装VMware Tools，然后进入linux
```bash
#创建一个文件夹，以挂载cdrom
mkdir /mnt/cdrom  
#你可以先去/dev目录下查看有没有cdrom这个设备，这一步是挂载cdrom到/mnt/cdrom
mount /dev/cdrom /mnt/cdrom  
cd /mnt/cdrom
#因为在/mnt/cdrom为挂载点。我们连root权限下也不能操作，所以复制出挂载点再操作
cp VMwareTools-10.0.5-3228253.tar.gz /mnt/VMwareTools-10.0.5-3228253.tar.gz 
cd /mnt
#解压操作不多说
tar -zxvf  VMwareTools-10.0.5-3228253.tar.gz 
cd  vmware-tools-distrib  #解压之后多出 vmware-tools-distrib这个文件夹，进去
./vmware-install.pl #安装，接着狂按回车就成功了。
```
tools安装完毕，重启以后共享文件夹不会丢失了。

#### 设置网络
正常默认为NAT模式,此模式下虚拟机上网没问题，win下也可以访问虚拟机中的服务，但是局域网内的主机无法访问我虚拟机中的服务，因为不在一个网段内。

现在需要把虚拟机置于和宿主机一个网段内。

1，打开win10的“网络设置”–>“更改适配器选项”

2，按住Ctrl，选中主机正在上网的网卡 和 “VMnet8” ，右键点击“桥接”，此时会创建出一个新的网络，我的名字叫“网桥”。

3，右键新的网络，点击“属性”，找到“IPv4”，打开其“属性”，默认为自动获取IP，改成手动设置，IP填写宿主机在局域网中的IP，子网掩码默认，在默认网关处填写路由器的IP，DNS 随意，如114.114.114.114 。

4，此时新的网络在搜索网络，如果能上网，会显示已连接到某某网络，打开网页测试。如果还不能上网，问题应该出在手动获取的IP上面。请仔细修改。

5，修改虚拟机的ip，将虚拟机的ip设置成和主机一个网段，比如 我的主机是192.168.99.50，linux 就设置成192.168.99.xx
```bash
vim /etc/network/interfaces #编辑配置文件
#加入以下项                    
auto eth33 #要设置的网卡
iface eth33 inet static #设置静态IP；如果是使用自动IP用dhcp，后面的不用设置，一般少用
address xxx.xxx.xxx.xxx #设置 和 window 在一个网段内的IP地址
netmask xxx.xxx.xxx.xxx #子网掩码
gateway xxx.xxx.xxx.xxx #网关
```
6，修改DNS
```bash
vim /etc/resolv.conf #设置这个文件重启后会覆盖，如果要持久的保存，需要修改：/etc/resolvconf/resolv.conf.d/base
#加入DNS配置
nameserver 8.8.8.8 #希望修改成的DNS
```
7，重启网络服务,先运行一次，然后在rc.local里加入这个重启网络配置的命令：
```bash
sudo /etc/init.d/networking restart #使网卡配置生效
sudo /etc/init.d/resolvconf restart #使DNS生效
```
大功告成。主机虚拟机都可以上网，而且都在相同的局域网段中，与其他局域网主机ping也可以ping通了。你的同事可以访问你虚拟机中的web服务了。

#### 构建启动docker容器

具体步骤见  [https://github.com/haixinz/docker-lnmp](https://github.com/haixinz/docker-lnmp).
