---
title: php实现守护进程
date: 2016-12-29 16:00:01
tags: php
---

#### 第一种方式，借助 nohup 和 &  配合使用。

在命令后面加上 & 符号， 可以让启动的进程转到后台运行，而不占用控制台，控制台还可以再运行其他命令，这里我使用一个while死循环来做演示，代码如下


```php
while(true){
    echo time().PHP_EOL;
    sleep(3);
}
```
用 & 方式来启动该进程

```bash
[root@localhost php]# php deadloop.php &
[1] 3454
[root@localhost php]# ps aux | grep 3454
root   3454 0.0 0.8 284544 8452 pts/0  T  18:06  0:00 php deadloop.php
root   3456 0.0 0.0 103316  896 pts/0  S+  18:08  0:00 grep 3454
[1]+ Stopped         php deadloop.php
[root@localhost php]#
```
可以看到该进程并未占用控制台，控制台还可以运行其他命令，这时我们还可以通过 fg 命令让进程恢复到普通占用控制台的模式。
```bash
[root@localhost php]# fg
php deadloop.php
1470996682
1470996685
1470996688
1470996691
```
以上就是关于 & 命令简单介绍。

下面再来看另一个命令 nohup

在命令之前加上 nohup ，启动的进程将会忽略linux的挂起信号 (SIGHUP)，那什么情况下会触发linux下SIGHUP信号呢，以下内容摘自百度百科：

SIGHUP会在以下3种情况下被发送给相应的进程：

1、终端关闭时，该信号被发送到session首进程以及作为job提交的进程（即用 & 符号提交的进程）
2、session首进程退出时，该信号被发送到该session中的前台进程组中的每一个进程
3、若父进程退出导致进程组成为孤儿进程组，且该进程组中有进程处于停止状态（收到SIGSTOP或SIGTSTP信号），该信号会被发送到该进程组中的每一个进程。

结合 1和2 我们知道，不管是否以 & (job方式)启动的进程，关闭终端时都会收到  SIGHUP 信号 ，那么进程收到 SIGHUP 信号会如何处理呢，看同样是摘自百度百科的一句话

系统对SIGHUP信号的默认处理是终止收到该信号的进程。所以若程序中没有捕捉该信号，当收到该信号时，进程就会退出。

也就是说关闭终端进程会收到SIGHUP信号，而该信号的默认处理方式就是结束掉该进程，当然 我们也可以自己处理该信号，或者忽略它，同样是上述循环的例子，我们稍加改进
```php

declare(ticks = 1);
pcntl_signal(SIGHUP, function(){
    // 这地方处理信号的方式我们只是简单的写入一句日志到文件中
    file_put_contents('logs.txt', 'pid : ' . posix_getpid() . ' receive SIGHUP 信号' . PHP_EOL);
});
while(true){
    echo time().PHP_EOL;
    sleep(3);
}
```

我们大可不必这么麻烦，只需要使用linux提供给我们的nohup命令，但我们使用nohup启动进程时，关闭终端，进程会忽略SIGHUP信号，也就不会退出了，首先去掉刚才的信号处理代码。然后nohup 运行。

```bash
[root@localhost php]# nohup php deadloop.php
```
nohup: 忽略输入并把输出追加到"nohup.out"

并且nohup默认会把程序的输出重定向到当前目录下的nohup.out文件，如果没有可写权限，则写入 $homepath/nohup.out
```bash
[root@localhost php]# ls
cmd.sh deadloop.php getPhoto.php nohup.out pics
[root@localhost php]# tail -f nohup.out
1470999772
1470999775
1470999778
1470999781
1470999784
1470999787
1470999790
1470999793
1470999796
1470999799
1470999802
```
此时 关闭终端，进程不会结束，而是变成了孤儿进程（ppid=1），因为创建它的父进程退出了。
```bash
[root@localhost ~]# ps -ef | grep 3554
root   3554 3497 0 19:09 pts/0  00:00:00 php deadloop.php
root   3575 3557 0 19:10 pts/1  00:00:00 grep 3554
[root@localhost ~]# ps -ef | grep 3554
root   3554   1 0 19:09 ?    00:00:00 php deadloop.php
root   3577 3557 0 19:10 pts/1  00:00:00 grep 3554
[root@localhost ~]#
```
结论： 所以当我们组合 nohup 和 & 两种方式时，启动的进程不会占用控制台，也不依赖控制台，控制台关闭之后进程被1号进程收养，成为孤儿进程，这就和守护进程的机制非常类似了。

```bash
[root@localhost php]# nohup php deadloop.php >logs.txt 2>error.txt &
[1] 3612
[root@localhost php]# ps -ef |grep 3612
root   3612 3557 0 19:18 pts/1  00:00:00 php deadloop.php
root   3617 3557 0 19:19 pts/1  00:00:00 grep 3612
[root@localhost php]#
```
其中 >logs.txt 重定向标准输出，2>error.txt 重定向标准错误输出。

#### 第二种实现方式就是根据守护进程的规则和特点通过代码来实现

守护进程最大的特点就是脱离了用户终端和会话，下面是实现的代码

```php

$pid = pcntl_fork();
if ($pid == -1)
{
  throw new Exception('fork子进程失败');
}
elseif ($pid > 0)
{
  //父进程退出,子进程变成孤儿进程被1号进程收养，进程脱离终端
  exit(0);
}
// 最重要的一步，让该进程脱离之前的会话，终端，进程组的控制
posix_setsid();
// 修改当前进程的工作目录，由于子进程会继承父进程的工作目录，修改工作目录以释放对父进程工作目录的占用。
chdir('/');
/*
 * 通过上一步，我们创建了一个新的会话组长，进程组长，且脱离了终端，但是会话组长可以申请重新打开一个终端，为了避免
 * 这种情况，我们再次创建一个子进程，并退出当前进程，这样运行的进程就不再是会话组长。
 */
$pid = pcntl_fork();
if ($pid == -1)
{
  throw new Exception('fork子进程失败');
}
elseif ($pid > 0)
{
  // 再一次退出父进程，子进程成为最终的守护进程
  exit(0);
}
// 由于守护进程用不到标准输入输出，关闭标准输入，输出，错误输出描述符
fclose(STDIN);
fclose(STDOUT);
fclose(STDERR);
/*
 * 处理业务代码
 */
while(TRUE)
{
  file_put_contents('log.txt', time().PHP_EOL, FILE_APPEND);
  sleep(5);
}
```
以上。
