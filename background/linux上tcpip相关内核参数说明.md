# linux内核网络相关参数意义及原理
> 本文只关注tcp/ip相关参数，默认值取自 ubuntu 16.04 LTS x64 桌面版

## /proc/sys/net/ipv4下tcp/ip专用参数
### 1. tcp 相关参数
参考  https://www.frozentux.net/ipsysctl-tutorial/chunkyhtml/tcpvariables.html

https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt
#### tcp 窗口、缓存相关参数

- tcp_window_scaling

意义:设置tcp/ip会话的滑动窗口大小是否启用放大机制

默认值: 1

相关tcp/ip原理:

	不开启窗口放大机制时，tcp的头部只有16bit空间来指名当前窗口大小，最大只能有65535.
	在目前高延迟、高带宽普遍的互联网（长肥管道），这个最大窗口大小是很不合适的。
	比如，你有个100Mbps链路，到www.t66y.com的延迟（rtt）为300ms。那么这条链路支持的最大性能是
	100M/8 *0.3=3.75MB/s
	但是最大窗口时65535时，能获得的最大性能是： 65535/0.3=213KB/s
	只有链路支持最大性能的 213k/3.75M = 6%， 利用率相当低。
	
	开启窗口放大机制时，链路双方在3次握手时，将协商放大倍数（最大为2^14倍 https://tools.ietf.org/html/rfc1323 ）。
	在双方都同意的前提下，协商出一个放大倍数（这个倍数在tcp头部的opt字段中指名，原来的16bit窗口值不变），然后在后续的所有报文中使用。
	假如协商的放大倍数为8，那么上述的链路最大窗口就可以达到65535*8=512KB，能获得最大性能为1.7MB/s，利用率48%。

- tcp_adv_win_scale

意义:配置缓存开销，有两种情况：1）tcp_adv_win_scale < =0，此时这个参数用来控制tcp窗口扩张时的额外开销 overhead=bytes(1-1/2^(-tcp_adv_win_scale)) 2）tcp_adv_win_scale > 0，此时这个参数用来控制tcp总读缓存中，可用于应用程序缓存的大小 size=tcp总缓存 / 2 ^ tcp_adv_win_scale

默认值: 1

相关tcp/ip原理:

	TCP的接收缓存中，除了被kernel用作接收窗口（存储TCP连接中的数据）外，还需要存储尚未被应用程序读取的tcp报文
	（应用程序可能因为各种原因没有来得及读取报文，必须将这些报文缓存起来）。
	tcp_adv_win_scale > 0时，用来控制此二者占总缓存的比例。
	tcp_adv_win_scale的值直接影响接收窗口大小，从而影响网络性能。
	比如，tcp_adv_win_scale=1 时，接收窗口大小为： 87380*（1-1/2^1)=43690;
	如果一个网络连接的RTT为150ms，则在这个网络连接上能够获得的最大性能为  43690/0.150=291267 B/s， 约为290kB/s,速度相当慢。
	因此为了获得更大的性能，必须尽可能的提高守护进程的处理能力，尽可能的减少应用程序缓存,
	从而扩大TCP接收窗口大小，使得TCP能够开足马力运行。


- tcp_app_win

意义:窗口中保留max(window/2^tcp_app_win, mss)字节用于应用缓冲。当为0时表示不需要保留。

默认值: 31

相关tcp/ip原理:
	
	对此参数意义存疑，意思是否是在tcp窗口中保留一些空间用来提供给tcp_adv_win_scale参数指定的应用缓存？

- tcp_mem

意义:针对整个tcp协议栈使用内存控制，包含三个值（单位为页，与操作系统相关），分别是

> low：当TCP使用了低于该值的内存页面数时，TCP不会考虑释放内存。

> presure：当TCP使用了超过该值的内存页面数量时，TCP试图稳定其内存使用，进入pressure模式，当内存消耗低于low值时则退出pressure状态。在pressure模式下，tcp协议栈将降低所有tcp连接的发送和接收缓存大小。

> high：最大内存使用量。如果TCP使用了这个数量的内存，将开始丢弃数据包，直到内存使用降低为止。

默认值: 44451	59269	88902

相关tcp/ip原理:


	
- tcp_wmem

意义:三个值，分别是

> Min：为每个TCP socket分配的用于发送缓冲的内存最小值。每个tcp socket都可以在建立后使用它。

> Default：为每个TCP socket预留用于发送缓冲的内存数量，该值会覆盖其它协议使用的net.core.wmem_default 值，并且要低于net.core.wmem_default的值。

> Max：用于每个TCP socket发送缓冲的内存最大值。

默认值: 4096	16384	4194304

相关tcp/ip原理:


- tcp_rmem

意义:三个值（单位为byte），分别是

> Min：为TCP socket预留用于接收缓冲的内存最小值。每个tcp socket都可以在建立后使用它。即使在内存出现紧张情况下每个tcp socket都至少会有这么多数量的内存用于接收缓冲.

> Default：为TCP socket预留用于接收缓冲的内存数量，默认情况下该值会影响其它协议使用的net.core.rmem_default 值，一般要低于net.core.rmem_default的值。

> Max：用于TCP socket接收缓冲的内存最大值。

默认值: 4096	87380	6291456

相关tcp/ip原理:
	
	tcp_rmem一般情况下不需要改动，tcp协议栈能够自动调整到最有效的值。在优化tcp性能时，改动tcp_mem, tcp_wmem即可。

- tcp_moderate_rcvbuf

意义:设置是否开启接收窗口大小自动调整机制

默认值: 1

相关tcp/ip原理:

	开启tcp接收窗口自动调整机制时，tcp将采用滑动窗口，自动调整窗口大小，使其最符合当前系统和网络环境，以达到最佳的性能。
	如果关闭自动调整机制，将导致窗口大小不变，影响性能。
	另外，在编程时，如果对socket使用
	setsockopt(s,SOL_SOCKET,SO_RCVBUF,(const char*)&nRecvBuf,sizeof(int));
	函数设置属性，将关闭窗口自动调整机制，一直维持指定窗口大小。
	
	滑动窗口移动规则：
	1、窗口合拢：
	在收到对端数据后，自己确认了数据的正确性，这些数据会被存储到缓冲区，等待应用程序获取。
	但这时候因为已经确认了数据的正确性，需要向对方发送确认响应ACK，又因为这些数据还没有被应用进程取走，这时候便需要进行窗口合拢，缓冲区的窗口左边缘向右滑动。
	注意响应包括：ACK序号（期望对方下一次发送数据包的序号），当前窗口大小。

	2、窗口张开：
	窗口收缩后，应用进程一旦从缓冲区中取出数据，TCP的滑动窗口需要进行扩张，这时候窗口的右边缘向右扩张，
	实际上窗口这是一个环形缓冲区，窗口的右边缘扩张会使用原来被应用进程取走内容的缓冲区。
	在窗口进行扩张后，需要使用ACK通知对端，这时候ACK的序号依然是上次确认收到包的序号。
	一个对方发送的序号，可能因为窗口张开会被响应（ACK）多次。

	3、窗口收缩：
	窗口的右边缘向左滑动，称为窗口收缩，Host Requirement RFC强烈建议不要这样做，但TCP必须能够在某一端产生这种情况时进行处理。
	
#### tcp拥塞控制
相关原理参考

http://www.blog.csdn.net/sicofield/article/details/9708383
	
http://www.blog.csdn.net/dog250/article/details/52962727


- tcp_allowed_congestion_control

意义:允许使用的拥塞控制算法，从提供值里面选择

默认值: cubic reno

相关tcp/ip原理:



- tcp_available_congestion_control

意义:tcp目前可以使用的拥塞控制算法

默认值: cubic reno

相关tcp/ip原理:

- tcp_congestion_control

意义:当前正在使用的拥塞控制算法

默认值: cubic

相关tcp/ip原理:


- tcp_fastopen

意义:

默认值: 1

相关tcp/ip原理:

	tcp fast open，就是在3次握手的时候也用来交换数据。
	产生这个需求的原因有2：
	1，3次握手至少造成一个rtt的延迟，降低了新建连接的响应速度
	2，如果采用keepalive机制，减少新建连接，会遇到keepalive的idle时间限制，超时的长连接会被中断，需要重新连接 （www.ietf.org/proceedings/80/slides/tcpm-3.pdf）。
	3，原始tcp协议支持在syn报文中携带数据，但是必须在3次握手之后。
	tfo的实现方式：
	服务器和客户端都支持tcp_fastopen，
	第一次握手时，服务器给客户端一个cookie，客户端记录此cookie，然后按照正常tcp流程运行。
	后续再次新建连接，需要握手时，客户端在syn包中包含记录的cookie，并携带数据；
	服务器验证cookie，发现客户端是之前连接过的合法客户端，就直接响应数据，无须3次握手。
	如下图：

![tfo](https://github.com/cyberhorse208/understandingngx/raw/master/background/tcp_fast_open.jpg)

	另外，linux 3.7.1以上支持tcp_fastopen, nginx 的listen指令也支持fastopen参数。


- tcp_slow_start_after_idle

意义: 链路上有一段时间没有包传输后，此参数控制是否重新进入慢启动

默认值: 1

相关tcp/ip原理:

	链路上空闲一段时间后，我们假设网络状况可能会有变化，因此重新开始慢启动。
	实际上，针对服务器来说，最好是设置为0，即关闭重新开始慢启动。
	因为对目前的互联网来说，短期内客户端与服务器之间的网络状况不会有很大变化，可以以之前的窗口大小直接开始传输。
	最多真正遇到拥塞时，进行拥塞避免即可。
	
- tcp_mtu_probing

意义:是否开启tcp层路径mtu发现，自动调整tcp窗口等

默认值: 0

相关tcp/ip原理:

	MTU: Max Transfer Unit, 链路层单个数据帧的最大值，超过这个长度的数据都要被分成多个帧进行传输。
	以太网中，MTU为1500。因此，在tcp协议里，最大的tcp数据分片长度为：
	MSS = MTU-IP Header - TCP Header = 1500 - 20 - 20 = 1460 B
	互联网上，一条链路可能需要经过很多节点，每个节点的MTU都有可能不一样，整条链路的MTU由最小的MTU决定。
	开启tcp路径发现，就是要发现链路最小的MTU，然后发送不超过这个MTU决定的MSS大小的数据包，
	这样能够避免IP层、链路层的分包，从而提高节点处理速度，进而提高整个链路的速度。

- tcp_probe_interval

意义:指定间隔多长时间进行一次路径MTU发现

默认值: 600

相关tcp/ip原理:

- tcp_probe_threshold

意义:定义何时停止路径MTU发现

默认值: 8

相关tcp/ip原理:


- tcp_pacing_ca_ratio

意义:

默认值: 120

相关tcp/ip原理:

- tcp_pacing_ss_ratio

意义:

默认值: 200

相关tcp/ip原理:


- tcp_recovery

意义:

默认值: 1

相关tcp/ip原理:


#### tcp 状态机相关参数
参考 http://www.jaminzhang.github.io/network/TCP-Finite-State-Machine/

![statmechine](https://github.com/cyberhorse208/understandingngx/raw/master/background/tcp_stat_mechine.jpeg)	


- tcp_fin_timeout

意义:本端主动断开的socket连接后，TCP保持在FIN-WAIT-2状态的时间。

默认值: 60

相关tcp/ip原理:

	当服务器怀疑对端因为某种原因已经无响应后，主动断开连接。那么服务器端的tcp状态将依次转换为
	FIN_WAIT_1 ---> FIN_WAIT_2 ---> TIME_WAIT
	在每个状态上都有相应的保持时间。在FIN_WAIT_2状态上的保持时间由tcp_fin_timeout指定。超时后进入下一状态。
	每个tcp连接FIN_WAIT_2状态上消耗内存1.5kb，如果服务器有大量FIN_WAIT_2状态连接，超时时间设置过长，会占用内存，而且更重要的是占用了端口。

- tcp_tw_recycle

意义:是否打开快速 TIME-WAIT sockets 回收

默认值: 0

相关tcp/ip原理:

- tcp_tw_reuse

意义:是否允许重新应用处于TIME-WAIT状态的socket用于新的TCP连接

默认值: 0

相关tcp/ip原理:

	tcp socket在处于TIME-WAIT状态2MSL时间后，才会转移到CLOSE状态。在此期间，这个socket还是占用原来的ip和端口。
	因此，如果在这个时间内想重用这个ip和端口，将会收到 “Address already in use“错误提示，
	在此期间，服务器程序重启会失败。
	
	tcp_tw_reuse 应该在连接的发起方使用，而不能在被连接方使用。
	举例来说：
	客户端向服务器端发起 HTTP 请求，服务端响应后主动关闭连接，
	于是 TIME_WAIT 便留在了服务端，此类情况使用「tcp_tw_reuse」是无效的，因为服务端是被连接方，所以不存在复用连接一说。
	比如说服务端是 PHP，它查询另一个 MySQL 服务端，然后主动断开连接，于是 TIME_WAIT 就落在了 PHP 一侧，
	此类情况下使用「tcp_tw_reuse」是有效的，因为此时 PHP 相对于 MySQL 而言是客户端，它是连接的发起方，所以可以复用连接。
	复用连接能够带来最明显的好处就是不用再次进行3次握手就能够传输数据，至少节约1个rtt时间。
	
- tcp_max_tw_buckets

意义: 系统在同时所处理的最大timewait sockets 数目。如果超过此数的话，time-wait socket 会被立即砍除并且显示警告信息

默认值: 16384

相关tcp/ip原理:



- tcp_keepalive_time

意义:表示连接上从最后一个包结束后多少秒内没有活动，就发送keepalive包保持连接

默认值: 7200

相关tcp/ip原理:



- tcp_keepalive_probes

意义:表示丢弃TCP连接前，进行最大TCP保持连接侦测的次数

默认值: 9

相关tcp/ip原理:

- tcp_keepalive_intvl

意义:当发送keepalive probe后，等待多长时间的响应。

默认值: 75


	
相关tcp/ip原理:

	一个连接上没有报文传输，等待tcp_keepalive_time 秒后，发送tcp_keepalive_probes个 keepalive probe，
	每个 keepalive probe之间等待tcp_keepalive_intvl秒，看是否有响应。如果所有 keepalive probe都发送完还是没有响应，则断开连接。
	
	
#### tcp连接相关参数

- tcp_abort_on_overflow

意义:是否在守护进程太忙而不能接受新的连接，就向对方发送reset消息

默认值: 0

相关tcp/ip原理:

	守护进程忙于做其他工作，来不及调用accept函数来接受新的连接请求，导致内核中TCP等待接受队列满。
	如果设置了此标志，则对新来的客户端连接请求（syn)直接发送reset消息。客户端就会表现为连不上服务器，体验较差。
	一般我们不能打开此标志，而是要着手于提高守护进程性能，从而能够接受更多的新连接，实在不行则需要扩充服务器。

- tcp_max_orphans

意义:系统所能处理不属于任何进程的TCP sockets最大数量。

默认值: 16384

相关tcp/ip原理:

	假如超过这个数量，那么不属于任何进程的连接会被立即reset，并同时显示警告信息。之所以要设定这个限制，纯粹为了抵御那些简单的 DoS 攻击，千万不要依赖这个或是人为的降低这个限制

- tcp_max_syn_backlog

意义:对于那些依然还未获得应用层accept的连接请求，需要保存在队列中最大数目

默认值: 128

相关tcp/ip原理:

	服务器绑定、监听了某个端口后，这个端口的SYN队列和ACCEPT队列就建立好了。
	客户端使用connect向服务器发起TCP连接，客户端的SYN包到达了服务器后，
	内核会把这一信息放到SYN队列（即未完成握手队列）中，同时回一个SYN+ACK包给客户端。
	一段时间后，在客户端再次发来了针对服务器SYN包的ACK网络分组时，内核会把连接从SYN队列中取出，
	再把这个连接放到ACCEPT队列（即已完成握手队列）中。
	服务器在调用accept时，其实就是直接从ACCEPT队列中取出已经建立成功的连接套接字而已。
	如下图所示：

![backlog](https://github.com/cyberhorse208/understandingngx/raw/master/background/tcp_backlog.jpeg)	


- tcp_orphan_retries

意义:针对孤立的socket(也就是已经从进程上下文中删除了，可是还有一些清理工作没有完成).在丢弃TCP连接之前重试的最大的次数

默认值: 0

相关tcp/ip原理:


- tcp_synack_retries

意义:对于远端的连接请求SYN，内核会发送SYN ＋ ACK数据报，以确认收到上一个 SYN连接请求包。这是所谓的三次握手.这里决定内核在放弃连接之前所送出的 SYN+ACK 数目

默认值: 5

相关tcp/ip原理:


#### tcp排序、重传参数

- tcp_reordering

意义:TCP流中重排序的数据报最大数量

默认值: 3

相关tcp/ip原理:


- tcp_max_reordering

意义:

默认值: 300

	tcp_reordering和tcp_max_reordering的值与快速重传机制有关。
	
	快速重传不依赖定时器的超时，而是依靠ACK确认包来进行重传。一旦认定出现了特定数量的重复ACK确认包，就认为需要进行重传。使用快速重传相比RTO超时重传通常可以更高效的修复TCP丢包问题。
	
	快速重传是基于一个前提：即按照RFC5681，当TCP收到一个乱序报文的时候应该立即回复ACK确认包，而不会延迟ACK(延迟ACK介绍参考之前文章介绍)确认。另外RFC5681还指出如果接收序列号空间存在洞，新接收的报文完全填充了这个洞或者部分填充了这个洞，TCP也应该立即回复一个ACK确认包以便发送端及时获取接收端相关的信息。

	 我们举个例子假设有5个TCP报文，P1(1-10)、P2(11-20)、P3(21-30)、P4(31-40)、P5(41-50)，其中括号中标注的是报文的比特系列号，每个报文的长度都为10bytes，发送端依次发送这5个报文。
	 	
	产生重复ACK报文的情况有3种：
	
	1. 报文丢失

	 假设P2报文在网络传输过程中丢失，P1、P3、P4、P5报文依次按序到达，接收端收到这P1的时候发送ack=11的确认包(实际上这里可能会延迟发送ACK报文，为了描述简单我们假设立即发送ACK报文)，接收端收到P3的时候发现是乱序的报文则会立即回复ack=11确认包，同样后面收到P4和P5的时候还是会回复ack=11的确认包。这样发送端就会连续接收到4个ack=11的确认包，后面三个确认包因为和第一个ack number重复，因为称呼为duplicate ACK。因此发送端就可以依据dup ACK来推测接收端的接收情况。
	 
	 2，报文乱序
	 
	 如果网络传输过程中发生乱序导致接收端接收顺序变为P1、P3、P2、P4、P5，这样的情况下也会产生一个dup ACK。
	 
	 3，IP层重复报文
	 
	 IP层重复提交了ACK报文给TCP层。
	 
	 因此，我们仅仅通过一个dup ACK并不能可靠的确认是发生了丢包还是发生了乱序传输，因此会存在一个门限(duplicate ACK threshold或者叫做dupthresh)，当TCP收到的dup ACK数超过这个门限的时候，就会认为发生了丢包，进而初始化一个快速重传。
	 
	 最初协议中给出的dupthresh这个门限是3，但是RFC 4653给出了一种调整dupthresh的方法。Linux中则可以通过**tcp_reordering**来设置默认值，另外Linux可能还会根据乱序测量的结果来更新实际的dupthresh。dupthresh的范围最终会在**tcp_reordering**和**tcp_max_reordering**之间。
	


- tcp_retrans_collapse

意义:是否开启重传重组包功能

默认值: 1

相关tcp/ip原理:


- tcp_retries1

意义:放弃回应一个TCP连接请求前进行重传的次数

默认值: 3

相关tcp/ip原理:

	根据rfc1122，存在一个阈值R1，当重传次数或者时间超过R1时，tcp向ip层发出“negative advice”，触发ip层的dead-gateway检测（MTU探测、路由刷新等）
	而R1的计算方式就由tcp_retries1指明。


- tcp_retries2

意义:放弃在已经建立通讯状态下的一个TCP数据包前进行重传的次数

默认值: 15

相关tcp/ip原理:

	linux内核会根据tcp_retries2计算出一个timeout，超过这个timeout秒后，不再进行数据包重传。
	计算方式为：
```c
// rto_base为200ms，TCP_RTO_MAX为120000ms

    		if (tcp_retries2 <= 9)
    			timeout = ((2 << tcp_retries2) - 1) * rto_base;
    		else
    			timeout = ((2 << 9) - 1) * rto_base +
    				(tcp_retries2 - 9) * TCP_RTO_MAX;
```

- tcp_syn_retries
	因此，如果数据包重传消耗时间已经超过了timeout时，即使还没有到tcp_retries2的次数，也停止重传。

意义:
	本机向外发起TCP SYN连接超时重传的次数，不应该高于255；该值仅仅针对外出的连接，对于进来的连接由tcp_retries1控制

默认值: 6

相关tcp/ip原理:
	

- tcp_sack

意义:是否启用有选择的应答（Selective Acknowledgment），这可以通过有选择地应答乱序接收到的报文来提高性能

默认值: 1

相关tcp/ip原理:
	
	如果发送端发送了5个报文p1,p2,p3,p4,p5，接收端只收到了p1,p3,p4,p5，那么接收端可以发送p1的ACK （累积确认，用于使发送端窗口右移），p3,p4,p5的SACK，告诉发送端p3,p4.p5已经收到，无须重传，可以释放他们占用的缓存。
	
	SACK在tcp的头部option中，最多能够一次 SACK 4个报文段。

	发送端维护一个未被确认的重传报文段队列，报文段未被确认前是不能释放的。重传送队列中的每个报文段都有一个标志位“SACKed”标识该报文段是否被SACK过，对于已经被SACK过的块，在重传时将被跳过。


- tcp_fack

意义:是否打开FACK快速重传功能

默认值: 1

相关tcp/ip原理:

	FACK（forward acknowledgement）是基于SACK的，FACK通过记录SACK块中系列号最大(forward-most)的SACK块来推测丢包信息。
	
	因为SACK块与ACK的值之间必然存在空洞，所以最大的SACK块与AC块之间的差值大于tcp_reordering个数据包的大小时，必然是ACK+1开始的数据包丢失了。
	
	启用FACK的最大好处是可以尽可能快的得知丢包信息。比如，不启用FACK的情况下，需要等待 tcp_reordering个重复的ACK包，才能确定发生丢包。
	
	例如：
	
	server端依次发出P1(0-9)、P2(10-19)、P3(20-29)、P4(30-39)、P5(40-49)，P6(50-59)。假设client端正常收到了P1包并回复了ACK确认包，P2、P3，P4，P5由于网络拥塞等原因丢失，client在收到P6时候，合并ACK后回复了一个Ack=10的确认包，并携带P6有SACK块信息（一个TCP头的option中最多4个SACK块），这样server在收到P1的确认包和P6的dup ACK时候，就可以根据dup ACK中的SACK信息得知client端收到了P1报文和P6报文，计算出P1和P6两个数据包中间间隔了4个数据包，达到了dup ACK门限(默认为3)，进而推测出P2报文丢失。
	
	
- tcp_frto

意义:设定RTO之后，观察多少个ACK，以确定真正执行重传。

默认值: 2

相关tcp/ip原理:

	F-RTO的基本思想是判断RTO是否正常，从而决定是否执行重传。方法是观察RTO之后的两个ACK。如果ACK不是冗余ACK，并且确认的包不是重传的，会认为RTO是虚假的就不执行重传。如果判断RTO是真实的，就进行重传（需要进入慢启动）。
	
	在传输链路RTT抖动比较大的场景下(例如无线网络)，FRTO可以有效的减少后续的虚假重传从而提升TCP性能。
	
	虚假重传(Spurious retransmission)产生场景包括：包传输中重排序、传输中发生包复制、ACK确认包传输中丢失等等。如果由于链路时延变化或者负载变化等因素导致RTT突然变大等原因，TCP的发送端可能还没收到ACK确认包就已经RTO超时而触发重传。
	
	虚假超时重传会降低网络性能，其主要由两方面的影响，一个是会导致重传已经发出的但是还没有收到ACK确认的数据，而这部分数据包的ACK报文可能只是延迟到达而已，因此对应的重传是不必要的，另一方面是虚假超时后进入慢启动阶段，每当收到一个ACK确认包的时候可以发出两个数据包，而原来初传的数据包实际上并没有丢失，因此增加了网络负载。

- tcp_early_retrans

意义:

默认值: 3

相关tcp/ip原理:

	ER （early retransmit）是在没有新数据可以发送的场景下降低快速重传dup ACK的门限，dup ACK是由乱序TCP报文触发的，但是发出的总数据包的个数少于4个的时候，就会因为没有足够的dup ACK而不能触发快速重传(假设默认dup ACK门限是3)。


- tcp_dsack

意义:是否使能DSACK

默认值: 1

相关tcp/ip原理:

	在发生虚假重传时，接收端接受到了重复的数据包，那么如何通知发送端不要再发送重复数据包呢？RFC2883定义了DSACK，用来通知发送端发生了虚假重传。
	
	DSACK基于SACK机制，通过使用SACK的第一个块来传递触发ACK确认包的序列号，无须额外的协商。发送端可以根据这个第一个块来推测是否发送了重复报文。
	
	接收端发送DSACK的处理逻辑：
	
	一个DSACK块只用来传递一个接收端最近接收到的重复报文的系列号，每个SACK选项中最多有一个DSACK块。每个重复包最多在一个DSACK块中上报一次。如果接收端依次发送了两个带有相同DSACK块信息的ACK报文，则表示接收端接收了两次重复包。
	
	如果收到重复报文，第一个SACK块应该应该指定触发这个ACK确认包的系列号。如果这个重复报文是一个大的不连续块的一部分，那么接下来的这个SACK块应该指定这个大的不连续块，额外的SACK块应该按照RFC2018指定的顺序排列。
	
	发送端接收到SACK报文时，处理逻辑：
	
	把第一个SACK块与这个ACK报文的ack number比较(而不是和当前已经接收到的最大的ack number比较)，如果小于等于ack number则说明是DSACK块，如果大于ack number则应该与第二个SACK块比较，如果第二个SACK块包含第一个SACK块，则说明第一个SACK块为DSACK块，如果上面两个条件都不满足说明第一个SACK块是普通的SACK块。

#### tcp 其他参数	
- tcp_base_mss

意义:

默认值: 1024

相关tcp/ip原理:

	MSS (Max Segment Size)， tcp最大分片大小，超过这个大小的数据都会被切分，然后以MSS的大小发送到网络。
	
	
- tcp_challenge_ack_limit

意义:

默认值: 1000

相关tcp/ip原理:



- tcp_ecn

意义:

默认值: 2

相关tcp/ip原理:

- tcp_ecn_fallback

意义:

默认值: 1

相关tcp/ip原理:


- tcp_fwmark_accept

意义:

默认值: 0

相关tcp/ip原理:

- tcp_invalid_ratelimit

意义:

默认值: 500

相关tcp/ip原理:




- tcp_l3mdev_accept

意义:

默认值: 0

相关tcp/ip原理:

- tcp_limit_output_bytes

意义:

默认值: 262144

相关tcp/ip原理:

- tcp_low_latency

意义:

默认值: 0

相关tcp/ip原理:


相关tcp/ip原理:


- tcp_min_rtt_wlen

意义:

默认值: 300

相关tcp/ip原理:

- tcp_min_tso_segs

意义:

默认值: 2

相关tcp/ip原理:


- tcp_no_metrics_save

意义:

默认值: 0

相关tcp/ip原理:

- tcp_notsent_lowat

意义:

默认值: -1

相关tcp/ip原理:



- tcp_rfc1337

意义: 这个开关可以启动对于在RFC1337中描述的"tcp 的time-wait暗杀危机"问题的修复。启用后，内核将丢弃那些发往time-wait状态TCP套接字的RST 包

默认值: 0

相关tcp/ip原理:


- tcp_stdurg

意义:使用 tcp urg pointer 字段中的主机请求解释功能

默认值: 0

相关tcp/ip原理:


- tcp_syncookies

意义:是否打开TCP同步标签(syncookie)，内核必须打开了 CONFIG_SYN_COOKIES项进行编译。同步标签(syncookie)可以防止一个套接字在有过多试图连接到达时引起过载

默认值: 1

相关tcp/ip原理:



- tcp_thin_dupack

意义:

默认值: 0

相关tcp/ip原理:

- tcp_thin_linear_timeouts

意义:

默认值: 0

相关tcp/ip原理:

- tcp_timestamps

意义:

默认值: 1

相关tcp/ip原理:

- tcp_tso_win_divisor

意义:控制根据拥塞窗口的百分比，是否来发送相应的延迟tso frame

默认值: 3

相关tcp/ip原理:


- tcp_workaround_signed_windows

意义:0：假定远程连接端正常发送了窗口收缩选项，即使对端没有发送.
1：假定远程连接端有错误,没有发送相关的窗口缩放选项

默认值: 0

相关tcp/ip原理:

### 2.udp 参数
- udp_mem

意义:三个值，分别是

> low：当UDP使用了低于该值的内存页面数时，UDP不会考虑释放内存。

> presure：当UDP使用了超过该值的内存页面数量时，UDP试图稳定其内存使用，进入pressure模式，当内存消耗低于low值时则退出pressure状态。

> high：允许所有UDP sockets用于排队缓冲数据报的页面量

默认值: 88902	118538	177804

原理:

- udp_rmem_min

意义:

默认值: 4096

原理:

- udp_wmem_min

意义:

默认值: 4096

相关tcp/ip原理:

- xfrm4_gc_thresh

意义:

默认值: 2147483647

原理:


### 3. ip层配置
- inet_peer_maxttl

意义:

默认值: 600

相关tcp/ip原理:

- inet_peer_minttl

意义:

默认值: 120

相关tcp/ip原理:

- inet_peer_threshold

意义:

默认值: 65664

相关tcp/ip原理:

- ip_default_ttl

意义:一个数据报的生存周期（Time To Live），即最多经过多少路由器

默认值: 64

相关tcp/ip原理:

- ip_dynaddr

意义:

默认值: 0

相关tcp/ip原理:

- ip_early_demux

意义:

默认值: 1

相关tcp/ip原理:

- ip_forward

意义:是否打开ipv4的IP转发

默认值: 0

相关tcp/ip原理:

- ip_forward_use_pmtu

意义:

默认值: 0

相关tcp/ip原理:

### ip数据报重组相关
- ipfrag_high_thresh

意义:表示用于重组IP分段的内存分配最高值

默认值: 4194304

相关tcp/ip原理:

- ipfrag_low_thresh

意义:表示用于重组IP分段的内存分配最低值

默认值: 3145728

相关tcp/ip原理:

- ipfrag_max_dist

意义:相同的源地址ip碎片数据报的最大数量. 这个变量表示在ip碎片被添加到队列前要作额外的检查.如果超过定义的数量的ip碎片从一个相同源地址到达，那么假定这个队列的ip碎片有丢失，已经存在的ip碎片队列会被丢弃

默认值: 64

相关tcp/ip原理:

- ipfrag_secret_interval

意义:ip碎片队列的重建延迟

默认值: 0

相关tcp/ip原理:

- ipfrag_time

意义:一个IP分段在内存中保留多少秒

默认值: 30

相关tcp/ip原理:

- ip_local_port_range

意义:本地发起连接时使用的端口范围，tcp初始化时会修改此值

默认值: 32768	60999

相关tcp/ip原理:

- ip_local_reserved_ports

意义:

默认值: 

相关tcp/ip原理:

- tcp_autocorking

意义:

默认值: 1

相关tcp/ip原理:

- ip_nonlocal_bind

意义:允许进程绑定到非本地地址

默认值: 0

相关tcp/ip原理:

- ip_no_pmtu_disc

意义:表示在全局范围内关闭路径MTU探测功能

默认值: 0

相关tcp/ip原理:

- ping_group_range

意义:

默认值: 1	0

相关tcp/ip原理:
### 4.cipso缓存配置
- cipso_cache_bucket_size

意义:限制cipso缓存项的数量，如果在缓存中新添加一行超出了这个限制，那么最旧的缓存项会被丢弃以释放出空间

默认值: 10

原理:

- cipso_cache_enable

意义:是否启用cipso缓存

默认值: 1

原理:

- cipso_rbm_optfmt

意义:是否开启cipso标志优化选项

默认值: 0

原理:

- cipso_rbm_strictvalid

意义:是否开启cipso选项的严格检测

默认值: 1

原理:


- fib_multipath_use_neigh

意义:

默认值: 0

原理:

- fwmark_reflect

意义:

默认值: 0

原理:

### 5. icmp相关配置
- icmp_echo_ignore_all

意义:

默认值: 0

原理:

- icmp_echo_ignore_broadcasts

意义:忽略所有接收到的icmp echo请求的广播

默认值: 1

原理:

- icmp_errors_use_inbound_ifaddr

意义:

默认值: 0

原理:

- icmp_ignore_bogus_error_responses

意义:

默认值: 1

原理:

- icmp_msgs_burst

意义:

默认值: 50

原理:

- icmp_msgs_per_sec

意义:

默认值: 1000

原理:

- icmp_ratelimit

意义:

默认值: 1000

原理:

- icmp_ratemask

意义:

默认值: 6168

原理:


### 6. igmp相关配置
- igmp_link_local_mcast_reports

意义:

默认值: 1

原理:

- igmp_max_memberships

意义:

默认值: 20

原理:

- igmp_max_msf

意义:

默认值: 10

原理:

- igmp_qrv

意义:

默认值: 2

原理:

