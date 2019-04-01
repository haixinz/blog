---
title: socket
date: 2016-12-29 09:44:32
tags: tcp/ip
---


## TCP/IP

TCP/IP（Transmission Control Protocol/Internet Protocol）即传输控制协议/网间协议，是一个工业标准的协议集，它是为广域网（WANs）设计的。

TCP/IP协议是一个协议簇。里面包括很多协议。比如 TCP协议，UDP协议，HTTP协议，IP协议，FTP协议....

TCP/IP 分为七层  物理层、数据链路层、网络层、运输层、会话层、表示层、应用层 (当年的大学考试题)，下层为上层提供接口

而上述提到的tcp/udp 协议属于运输层，http协议则属于应用层，http就应用到了tcp协议

http协议就不用说了。。准备耐心看完《http权威指南》

这里着重说一下TCP 和 UDP 协议的区别

TCP的优点： 可靠，稳定 TCP的可靠体现在TCP在传递数据之前，会有三次握手来建立连接，而且在数据传递时，有确认、窗口、重传、拥塞控制机制，在数据传完后，还会断开连接用来节约系统资源。 TCP的缺点： 慢，效率低，占用系统资源高，易被攻击 TCP在传递数据之前，要先建连接，这会消耗时间，而且在数据传递时，确认机制、重传机制、拥塞控制机制等都会消耗大量的时间，而且要在每台设备上维护所有的传输连接，事实上，每个连接都会占用系统的CPU、内存等硬件资源。 而且，因为TCP有确认机制、三次握手机制，这些也导致TCP容易被人利用，实现DOS、DDOS、CC等攻击。

UDP的优点： 快，比TCP稍安全。UDP没有TCP的握手、确认、窗口、重传、拥塞控制等机制。UDP是一个无状态的传输协议，所以它在传递数据时非常快。没有TCP的这些机制，UDP较TCP被攻击者利用的漏洞就要少一些。但UDP也是无法避免攻击的，比如：UDP Flood攻击…… 

UDP的缺点： 不可靠，不稳定 因为UDP没有TCP那些可靠的机制，在数据传递时，如果网络质量不好，就会很容易丢包。

基于上面的优缺点，什么时候应该使用TCP？ 当对网络通讯质量有要求的时候，
比如：整个数据要准确无误的传递给对方，这往往用于一些要求可靠的应用，比如HTTP、HTTPS、FTP等传输文件的协议，POP、SMTP等邮件传输的协议。 在日常生活中，常见使用TCP协议的应用如下： 浏览器，用的HTTP ；FlashFXP，用的FTP ；Outlook，用的POP、SMTP；Putty，用的Telnet、SSH 

什么时候应该使用UDP： 当对网络通讯质量要求不高的时候，要求网络通讯速度能尽量的快，这时就可以使用UDP。 
比如，日常生活中，常见使用UDP协议的应用如下： 语音通话，视频通话，视频直播 ……

## socket 

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的
TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。简单的说，soket 是应从层往下所有协议的api。



## 安装php socket 扩展

windows 下 ：添加 php_sockets.dll 到 php安装目录/ext 下

在配置文件中添加 extension=php_sockets.dll （或取消注释）

注：cli模式下运行的php版本是环境变量下设置的php版本。这里用集成环境可以切换php版本的朋友可以注意一下，否则可能会遇到添加扩展的版本和cli下运行的版本不符。

linux 下：

```bash
cd ext/sockets
/usr/local/php/bin/phpize
./configure --prefix=/usr/local/php/lib --with-php-config=/usr/local/php/bin/php-config --enable-sockets
make && make install
```
