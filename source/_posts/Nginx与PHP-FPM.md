---
title: Nginx与PHP-FPM
date: 2016-12-29 09:44:32
tags: nginx
---

## CGI

CGI全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。

CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如php,perl,tcl等。
CGI 是为了保证 web server 传递过来的数据是标准格式的，方便 CGI 程序的编写者。web server（比如说 nginx）只是内容的分发者。
当 web server 收到 /index.php 这个请求后，会启动对应的 CGI 程序，这里就是 PHP 的解析器。接下来 PHP 解析器会解析 php.ini 文件，初始化执行环境，然后处理请求，再以 CGI 规定的格式返回处理后的结果，退出进程。web server 再把结果返回给浏览器。
CGI 是个协议，跟进程什么的没关系。

## FastCGI
fastCgi 是用来提高 CGI 程序性能的。

FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次（这是CGI最为人诟病的fork-and-execute 模式）。它还支持分布式的运算，即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。

FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。

具体是怎么做到的呢？首先，fastCgi 会先启一个 master，解析配置文件，初始化执行环境，然后再启动多个 worker。当请求过来时，master 会传递给一个 worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当 worker 不够用时，master 可以根据配置预先启动几个 worker 等着；当然空闲 worker 太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是 fastCgi 对进程的管理。

- FastCGI的特点

1，FastCGI具有语言无关性.
2，FastCGI在进程中的应用程序，独立于核心web服务器运行，提供了一个比API更安全的环境。APIs把应用程序的代码与核心的web服务器链接在一起，这意味着在一个错误的API的应程序可能会损坏其他应用程序或核心服务器。 恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥。
4，FastCGI技术目前支持语言有：C/C++、Java、Perl、Tcl、Python、SmallTalk、Ruby等。相关模块在Apache, ISS, Lighttpd等流行的服务器上也是可用的。
5，FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变。

- FastCGI的工作原理

1，Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module),
2，FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
3，当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
4，FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。

- FastCGI的不足

因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。
Nginx 0.8.46+PHP 5.2.14(FastCGI)服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M * 10=150M），开启的64个php-cgi进程消耗1280M内存（20M*64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M。

- PHP-CGI

PHP-CGI是PHP自带的FastCGI管理器。
PHP-CGI的不足：
php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启。
直接杀死php-cgi进程，php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题，守护进程会平滑从新生成新的子进程。）


- PHP-FPM

PHP-FPM 是一个实现了 FastCgi 的程序，被 PHP 官方收录。

PHP 的解释器是 php-cgi，它只是个 CGI 程序，只能解析请求，返回结果，不会进程管理。所以就出现了一些能够调度 php-cgi 进程的程序，比如说由 lighthttpd 分离出来的 spawn-fcgi。PHP-FPM 也是这么个东西，在长时间的发展后，逐渐得到了大家的认可，也越来越流行。


## PHP-FPM FASTCGI 和 PHPCGI

有的说，php-fpm是fastcgi进程的管理器，用来管理fastcgi进程的。

对。php-fpm的管理对象是php-cgi。但不能说php-fpm是fastcgi进程的管理器，因为前面说了fastcgi是个协议，似乎没有这么个进程存在，就算存在php-fpm也管理不了他（至少目前是）。 

有的说，php-fpm是php内核的一个补丁。

以前是对的。因为最开始的时候php-fpm没有包含在PHP内核里面，要使用这个功能，需要找到与源码版本相同的php-fpm对内核打补丁，然后再编译。后来PHP内核集成了PHP-FPM之后就方便多了，使用--enalbe-fpm这个编译参数即可。

有的说，修改了php.ini配置文件后，没办法平滑重启，所以就诞生了php-fpm。

是的，修改php.ini之后，php-cgi进程的确是没办法平滑重启的。php-
fpm对此的处理机制是新的worker用新的配置，已经存在的worker处理完手上的活就可以歇着了，通过这种机制来平滑过度。

还有的说PHP-CGI是PHP自带的FastCGI管理器，那这样的话干吗又弄个php-fpm出来？

不对。php-cgi只是解释PHP脚本的程序而已。
## Nginx 与 PHP-FPM

nginx是web服务器，php-fpm是一个PHPFastCGI进程管理器，两者遵循fastcgi的协议进行通信，nginx负责静态类似
html文件的处理。

php-fpm负责php脚本语言的执行，这么设计的目的是为了解耦前端nginx和后端的php，不至于让容易出问题的php脚本堵
塞整个nginx的业务处理，影响用户体验，因为php脚本语言的执行是会比较容易出问题的。

nginx之所以能处理成千上万高并发业务，除其本身的异步非阻塞模式，在与和其他模块的耦合扩展方法也是分不开的，在nginx的设计里不能接受的就是阻塞，不过并非完全没有梗，比如说用到的最多的多进程单线程的模式，由于nginx日志没有单独的处理进程，如果收集日志时处理不当就会把worker进程堵死。

Nginx 是非阻塞IO & IO复用模型，通过操作系统提供的类似 epoll 的功能，可以在一个线程里处理多个客户端的请求。

Nginx 的进程就是线程，即每个进程里只有一个线程，但这一个线程可以服务多个客户端。

PHP-FPM 是阻塞的单线程模型，pm.max_children 指定的是最大的进程数量，pm.max_requests 指定的是每个进程处理多少个请求后重启(因为 PHP 偶尔会有内存泄漏，所以需要重启)。

PHP-FPM 的每个进程也只有一个线程，但是一个进程同时只能服务一个客户端。

## Nginx

### nginx的工作简介
 
接到php的脚本请求后，nginx通过fastcgi_pass指令将请求传递给后端php-fpm的worker进程处理，在此过程中，nginx做了各种超时机制、缓存机制、buffer机制和长连接机制等来保障与后端的php-fpm能够良性高效的合作。

正常执行中的nginx会有多个进程，最基本的有master process（监控进程，也叫做主进程）和woker process（工作进程），还可能有cache相关进程。

### nginx 设置

worker_processes 一般设置成cpu的个数，如果是多核的,一般也算作一块cpu,所以个数应该是 cpu个数 * 核数
。

## PHP-FPM

Php-fpm是一个PHPfastcgi进程管理器，在启动后会有master和worker两种进程，master负责接收外部信号和管理worker进程。worker进程是负责干活的，处理nginx传过来的任务。master进程只有一个，负责监听端口和管理worker进程，每次传来任务，与前端的nginx建立3次握手后放入连接队列，供worker进程进行accept，当worker进程出现错误或执行超时时，负责将worker进程重启或者杀掉，是php-fpm模型中的大内总管。

Worker进程是工作进程，每个worker进程都独立的执行php程序脚本，然后把执行的结果通过fastcgi协议交给nginx，执行过程中受master的管理。在工作中，worker进程去竞争accept管理进程master的链接队列，accept函数将从连接请求队列中获得连接信息，创建新的socket，并返回该套接字的fd，新创建的socket用于服务器与nginx的通信，而原来的套接字仍然处于监听状态。

php-fpm可以配置多个pool，所有pool由master统一管理监听不同端口并分配不同worker进程池，worker进程池支持动态prefork同时也支持静态开启，服务器内存较大时建议直接计算后配置静态资源池，可以减少频繁prefork进程所带来的开销，提高服务质量，由于进程模型越跑程序耗费越大，每个worker进程可以配置执行多少个请求后进行重启。

### php-fpm 设置

#### pm = static | dynamic | ondemand :

static : 静态模式，启动时分配固定的worker进程。 表示在fpm运行时直接fork出pm.max_chindren个worker进程，服务器内存比较大的可以用此模式，会节省出很多fork进程所带来的开销。

dynamic: 动态模式，启动时分配固定的进程。伴随着请求数增加，在设定的浮动范围调整worker进程。表示运行时fork出start_servers个进程，随着负载的情况，动态的调整，最多不超过max_children个进程。

ondemand: 按需分配，当收到用户请求时fork worker进程。

pm.max_children：静态方式下开启的php-fpm进程数量
pm.start_servers：动态方式下的起始php-fpm进程数量
pm.min_spare_servers：动态方式下的最小php-fpm进程数
pm.max_spare_servers：动态方式下的最大php-fpm进程数量

如果dm设置为 static，那么其实只有pm.max_children这个参数生效。系统会开启设置数量的php-fpm进程。
如果dm设置为 dynamic，那么pm.max_children参数失效，后面3个参数生效。系统会在php-fpm运行开始 的时候启动pm.start_servers个php-fpm进程，然后根据系统的需求动态在pm.min_spare_servers和pm.max_spare_servers之间调整php-fpm进程数。

#### 最大子进程数max_childen

原则上是越大越好，php-cgi的进程多了就会处理的很快，排队的请求就会很少。
一般来说一台服务器正常情况下每一个php-cgi所耗费的内存在20M左右 （一般算上内存泄漏算30M。假设“max_children”设置成100个，20M*100=2000M，也就是说在峰值的时候所有PHP-CGI所耗内存在2000M以内。假设“max_children”设置的较小，比如5-10个，那么php-cgi就会“很累”，处理速度也很慢，等待的时间也较长。如果长时间没有得到处理的请求就会出现504 Gateway Time-out这个错误，而正在处理的很累的那几个php-cgi如果遇到了问题就会出现502 Bad gateway这个错误。

#### 最大请求数max_requests

最大处理请求数是指一个php-fpm的worker进程在处理多少个请求后就终止掉，master进程会重新respawn一个新的。这个配置的主要目的是避免php解释器或程序引用的第三方库造成的内存泄露。 

配置举例：如web服务的机器是12核cpu、16G内存，nginx开启12个worker进程，php开启256个进程，跑起来后每个进程大概占用30M内存，也就是（256+12）*30=8G ，另外还跑了一些配管、监控、统计、日志收集等七七八八的软件，整体业务是比较轻松的。

### php-fpm进程管理方式

usr/local/php/sbin/php-fpm{start|stop|quit|restart|reload|logrotate}

start 启动php的fastcgi进程
stop 强制终止php的fastcgi进程
quit 平滑终止php的fastcgi进程
restart 重启php的fastcgi进程
reload 重新平滑加载php的php.ini
logrotate 重新启用log文件

## 502 与 504

### nginx502

#### 第一种情况

php-fpm的worker进程执行php程序脚本时，超过了配置的最长执行时间，master进程将worker进程杀掉，直接返回502。返回502后nginx对应的error日志是104: Connection reset by peer,对应的php执行时间的配置如下，一些版本中php-fpm的配置会覆盖php.ini的配，使php.ini的配置不起作用：

php.ini中默认30s：max_execution_time =
php-fpm中：request_terminate_timeout =

#### 第二种情况

连接请求数（accpet之前）超出了端口所能监听的tcp连接的最大值（backlog的值），进不了fpm等待accept的链接队列，直接返回502，这里可能会产生tcp重传；返回502后nginx对应的error日志是111: Connection refused, backlog的值是半连接和全连接的总和，他的存在也有短时间缓冲解耦nginx请求与fpm处理的作用，半连接指收到了syn请求，3次握手尚未建立，全连接指的是3次握手已经成功，不过尚未被accpet的请求，fpm里面有调节的参数，如果fpm的参数设置为-1，则默认走的是系统内核参数net.core.somaxconn的设置值，如果不设置可以在/proc/sys/net/core/somaxconn里查看，默认值是128，所以在连接请求较高的业务里要增大这个值。

#### 第三种情况

网络卡时，客户端断开连接，nginx处显示499，然后php检查到前端nginx产生abort后，又master结束此条任务的继而产生502，一般此种情况的报警，先是499，过会儿变成502，再过一会变成504。

#### 减少避免502报错优化建议

502主要从php-fpm的配置方考虑，根据服务器情况，适量增大php-fpm的工作进程数，适当增加php的执行时间，适当增加backlog值。

php的工作进程数也不是越大越好，这种进程模型运行时间长了占的内存会增大，一般一个php进程是占到30M左右的内存，开多少合适自己算吧，nginx的worker进程一般也能跑到30M的内存，综合计算一下；php的执行时间可以根据你的服务标准来设定，超过服务时间浏览器返回的是502错误，这个按照实际的情况处理吧，反正我是觉得执行的慢有返回结果总比直接返回502错误的强；至于backlog值，当程序写的比较好时，建议设置其数量为php工作进程的1到2倍。

### nginx日志里产生504错误

#### 第一种情况

php的worker进程池处理慢，无法尽快处理等待accept的链接队列，导致3次握手后的链接队列长时间没有被accept，nginx链接等待超时；返回504后nginx对应的error日志是110: Connection timed out。

#### 第二种情况

后端php-fpm执行脚本的时间太长，超过了nginx配置的超时机制，这个时候也是会报出504错误的。

#### 第三种情况

客户端的网络及其差，php将请求处理完交给nginx后，nginx没能在超时时间内将内容全部吐给用户，这时也会超时，只有504而没有502。

#### 减少避免504报错的优化建议

504主要从nginx的配置方考虑，根据业务情况配置好超时的各种机制，包含但不限于下属参数：

fastcgi_connect_timeout
fastcgi_send_timeout
fastcgi_read_timeout

另外：在配置过程中，比如遇到大并发或者是特殊业务的场景，不合理的fd、buffer等设置也会带来5XX错误，比如说大并发连接的业务要增大系统和单个程序的fd数量，如果是上传业务要增大头buffer等，这些要视情况而做优化，正所谓道法自然，术变万千，要以不变应万变。