# High Performance Browser Networking

本文是 https://hpbn.co 的读书笔记关于tcp性能优化的checklist。


## Performance Checklist


Optimizing TCP performance pays high dividends, regardless of the type of application, for every new connection to your servers. A short list to put on the agenda:

* Upgrade server kernel to latest version.

> 最新的内核通常包含了针对tcp的最新优化算法，直接使用最新内核可能会带来比较大的性能优化。


*    Ensure that cwnd size is set to 10.

> 拥塞控制窗口cwnd，是发送端根据网络状况进行的自我限制；在当前带宽较高、延迟大的环境中，适宜将 cwnd的初始值设置的比较大（10 是google的经验设置）。对于网页文本浏览、小图传输来说，尤其有用。就是能够尽可能快速的发送完，而不是逐步增大窗口，等待ack。linux 2.6.39开始包含initcwnd=10.

使用大的初始cwnd可以带来的好处示意图如下：

-- 使用小的初始cwnd传输一个64KB文件，需要264ms，过程如此：

![small-cwnd-init](https://github.com/cyberhorse208/understandingngx/raw/master/background/fetchfile-with-slow-start-small-inicwnd.png)


-- 使用初始cwnd=10传输，只需要96ms，过程如下：

![large-cwnd-init](https://github.com/cyberhorse208/understandingngx/raw/master/background/fetchfile-with-slow-start-large-inicwnd.png)


*    Ensure that window scaling is enabled.

> 这个不用多说，使用大窗口发送更多数据。可以参考  https://github.com/cyberhorse208/understandingngx/blob/master/background/linux上tcpip相关内核参数说明.md  中间关于tcp_window_scaling的讨论。


*    Disable slow-start after idle.

> 禁用 tcp_slow_start_after_idle。 可以参考 https://github.com/cyberhorse208/understandingngx/blob/master/background/linux上tcpip相关内核参数说明.md  中间关于tcp_slow_start_after_idle的讨论。

*    Investigate enabling TCP Fast Open.

> 针对幂等操作（GET请求等多次请求结果不便的请求）打开 tcp_fastopen。 可以参考 https://github.com/cyberhorse208/understandingngx/blob/master/background/linux上tcpip相关内核参数说明.md  中间关于tcp_fastopen的讨论。

*    Eliminate redundant data transfers.

> 从应用程序入手，尽量减少需要在网络上传输的数据。毕竟传输再快也快不过不传输。

*    Compress transferred data.

> 压缩传输的数据。

*    Position servers closer to the user to reduce roundtrip times.

> 离客户端更近些，比如使用CDN。

*    Reuse TCP connections whenever possible.

> 重用tcp连接，减少新建连接的需求，毕竟新建连接要3次握手，浪费时间
> 可用方式为启用keepalive，重用处于TIME-WAIT状态的连接等
> 参考 https://github.com/cyberhorse208/understandingngx/blob/master/background/linux上tcpip相关内核参数说明.md  中间关于tcp_tw_reuse、tcp_keepalive_time的讨论。


### "TCP Tuning for HTTP" recommendations. 
https://tools.ietf.org/html/draft-stenberg-httpbis-tcp-03

只列出上文没有说明的推荐设置。

* 提高系统支持的openfile数量

* 增大网卡数据包队列大小

> 尤其在网卡速度快于内核处理数据包的速度的时候

* 增大backlog队列大小

> 应对在应用层来不及处理新连接的情况下，缓存新连接等待处理。


* 增大local ports范围

> 这个是针对客户端，或者连接较多上游服务器的服务器来说的。

* 增大tcp协议栈能够使用的缓存

> 更大的缓存，意味着大的窗口，更多的连接，更高的性能。当然，前提是系统能够支持。

* 关闭 nagle算法

> nagle算法目的时尽量减少网络上小包的传输，因此会在发送小包时，稍微延迟等待后续包，以合并传输。
> 在现在的互联网环境下，nagle算法并不合适；
> http/1.1下，会影响使用 write() 系统调用发送一个完整请求的场景
> http/2 下，有关键的小包不能被延迟发送，因此nagle会严重影响其性能。

* 关闭延迟确认
> 一般情况下要关闭
> TLS 握手过程可以开启延迟确认

