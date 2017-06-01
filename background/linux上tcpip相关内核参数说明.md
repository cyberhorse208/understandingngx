# linux内核网络相关参数意义及原理
> 本文只关注tcp/ip相关参数，默认值取自 ubuntu 16.04 LTS x64 桌面版

## /proc/sys/net/ipv4下tcp/ip专用参数
### 1. tcp 相关参数
- tcp_abort_on_overflow
意义:是否在守护进程太忙而不能接受新的连接，就向对方发送reset消息
默认值: 0
相关tcp/ip原理:

- tcp_adv_win_scale
意义:
默认值: 1
相关tcp/ip原理:

- tcp_allowed_congestion_control
意义:允许使用的拥塞控制算法，从提供值里面选择
默认值: cubic reno
相关tcp/ip原理:

- tcp_app_win
意义:保留max(window/2^tcp_app_win, mss)数量的窗口用于应用缓冲。当为0时表示不需要缓冲
默认值: 31
相关tcp/ip原理:

- tcp_autocorking
意义:
默认值: 1
相关tcp/ip原理:

- tcp_available_congestion_control
意义:tcp目前可以使用的拥塞控制算法
默认值: cubic reno
相关tcp/ip原理:

- tcp_base_mss
意义:
默认值: 1024
相关tcp/ip原理:

- tcp_challenge_ack_limit
意义:
默认值: 1000
相关tcp/ip原理:

- tcp_congestion_control
意义:当前正在使用的拥塞控制算法
默认值: cubic
相关tcp/ip原理:

- tcp_dsack
意义:是否允许TCP发送“两个完全相同”的SACK
默认值: 1
相关tcp/ip原理:

- tcp_early_retrans
意义:
默认值: 3
相关tcp/ip原理:

- tcp_ecn
意义:
默认值: 2
相关tcp/ip原理:

- tcp_ecn_fallback
意义:
默认值: 1
相关tcp/ip原理:

- tcp_fack
意义:是否打开FACK拥塞避免和快速重传功能
默认值: 1
相关tcp/ip原理:

- tcp_fastopen
意义:
默认值: 1
相关tcp/ip原理:

- tcp_fin_timeout
意义:本端断开的socket连接，TCP保持在FIN-WAIT-2状态的时间。对方可能会断开连接或一直不结束连接或不可预料的进程死亡。如果您的机器为负载很重的web服务器，您可能要冒内存被大量无效数据报填满的风险，FIN-WAIT-2 sockets 的危险性低于 FIN-WAIT-1，因为它们最多只吃 1.5K 的内存，但是它们存在时间更长。
默认值: 60
相关tcp/ip原理:

- tcp_frto
意义:
默认值: 2
相关tcp/ip原理:

- tcp_fwmark_accept
意义:
默认值: 0
相关tcp/ip原理:

- tcp_invalid_ratelimit
意义:
默认值: 500
相关tcp/ip原理:

- tcp_keepalive_intvl
意义:
默认值: 75
相关tcp/ip原理:

- tcp_keepalive_probes
意义:表示丢弃TCP连接前，进行最大TCP保持连接侦测的次数
默认值: 9
相关tcp/ip原理:

- tcp_keepalive_time
意义:表示从最后一个包结束后多少秒内没有活动，才发送keepalive包保持连接，默认7200s，理想可设为1800s，即如果非正常断开，1800s后可通过keepalive知
默认值: 7200
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

- tcp_max_orphans
意义:系统所能处理不属于任何进程的TCP sockets最大数量。假如超过这个数量，那么不属于任何进程的连接会被立即reset，并同时显示警告信息。之所以要设定这个限制，纯粹为了抵御那些简单的 DoS 攻击，千万不要依赖这个或是人为的降低这个限制
默认值: 16384
相关tcp/ip原理:

- tcp_max_reordering
意义:
默认值: 300
相关tcp/ip原理:

- tcp_max_syn_backlog
意义:对于那些依然还未获得应用层accept的连接请求，需要保存在队列中最大数目
默认值: 128
相关tcp/ip原理:

- tcp_max_tw_buckets
意义: 系统在同时所处理的最大timewait sockets 数目。如果超过此数的话，time-wait socket 会被立即砍除并且显示警告信息
默认值: 16384
相关tcp/ip原理:

- tcp_mem
意义:该文件保存了三个值，分别是
> low：当TCP使用了低于该值的内存页面数时，TCP不会考虑释放内存。
> presure：当TCP使用了超过该值的内存页面数量时，TCP试图稳定其内存使用，进入pressure模式，当内存消耗低于low值时则退出pressure状态。
> high：允许所有tcp sockets用于排队缓冲数据报的页面量

默认值: 44451	59269	88902
相关tcp/ip原理:

- tcp_min_rtt_wlen
意义:
默认值: 300
相关tcp/ip原理:

- tcp_min_tso_segs
意义:
默认值: 2
相关tcp/ip原理:

- tcp_moderate_rcvbuf
意义:
默认值: 1
相关tcp/ip原理:

- tcp_mtu_probing
意义:是否开启tcp层路径mtu发现，自动调整tcp窗口等
默认值: 0
相关tcp/ip原理:

- tcp_no_metrics_save
意义:
默认值: 0
相关tcp/ip原理:

- tcp_notsent_lowat
意义:
默认值: -1
相关tcp/ip原理:

- tcp_orphan_retries
意义:针对孤立的socket(也就是已经从进程上下文中删除了，可是还有一些清理工作没有完成).在丢弃TCP连接之前重试的最大的次数
默认值: 0
相关tcp/ip原理:

- tcp_pacing_ca_ratio
意义:
默认值: 120
相关tcp/ip原理:

- tcp_pacing_ss_ratio
意义:
默认值: 200
相关tcp/ip原理:

- tcp_probe_interval
意义:
默认值: 600
相关tcp/ip原理:

- tcp_probe_threshold
意义:
默认值: 8
相关tcp/ip原理:

- tcp_recovery
意义:
默认值: 1
相关tcp/ip原理:

- tcp_reordering
意义:TCP流中重排序的数据报最大数量
默认值: 3
相关tcp/ip原理:

- tcp_retrans_collapse
意义:
默认值: 1
相关tcp/ip原理:

- tcp_retries1
意义:放弃回应一个TCP连接请求前进行重传的次数
默认值: 3
相关tcp/ip原理:

- tcp_retries2
意义:放弃在已经建立通讯状态下的一个TCP数据包前进行重传的次数
默认值: 15
相关tcp/ip原理:

- tcp_rfc1337
意义: 这个开关可以启动对于在RFC1337中描述的"tcp 的time-wait暗杀危机"问题的修复。启用后，内核将丢弃那些发往time-wait状态TCP套接字的RST 包
默认值: 0
相关tcp/ip原理:

- tcp_rmem
意义:三个值，分别是
> Min：为TCP socket预留用于接收缓冲的内存最小值。每个tcp socket都可以在建立后使用它。即使在内存出现紧张情况下tcp socket都至少会有这么多数量的内存用于接收缓冲
> Default：为TCP socket预留用于接收缓冲的内存数量，默认情况下该值会影响其它协议使用的net.core.rmem_default 值，一般要低于net.core.rmem_default的值。该值决定了在tcp_adv_win_scale、tcp_app_win和tcp_app_win=0默认值情况下，TCP窗口大小为65535。
> Max：用于TCP socket接收缓冲的内存最大值。该值不会影响net.core.rmem_max，"静态"选择参数SO_SNDBUF则不受该值影响。 

默认值: 4096	87380	6291456
相关tcp/ip原理:

- tcp_sack
意义:是否启用有选择的应答（Selective Acknowledgment），这可以通过有选择地应答乱序接收到的报文来提高性能（这样可以让发送者只发送丢失的报文段）；（对于广域网通信来说）这个选项应该启用，但是这会增加对 CPU 的占用
默认值: 1
相关tcp/ip原理:

- tcp_slow_start_after_idle
意义: 如果设置满足RFC2861定义的行为，在从新开始计算拥塞窗口前延迟一些时间，这延迟的时间长度由当前rto决定.
默认值: 1
相关tcp/ip原理:

- tcp_stdurg
意义:使用 TCP urg pointer 字段中的主机请求解释功能
默认值: 0
相关tcp/ip原理:

- tcp_synack_retries
意义:对于远端的连接请求SYN，内核会发送SYN ＋ ACK数据报，以确认收到上一个 SYN连接请求包。这是所谓的三次握手.这里决定内核在放弃连接之前所送出的 SYN+ACK 数目
默认值: 5
相关tcp/ip原理:

- tcp_syncookies
意义:是否打开TCP同步标签(syncookie)，内核必须打开了 CONFIG_SYN_COOKIES项进行编译。同步标签(syncookie)可以防止一个套接字在有过多试图连接到达时引起过载
默认值: 1
相关tcp/ip原理:

- tcp_syn_retries
意义:本机向外发起TCP SYN连接超时重传的次数，不应该高于255；该值仅仅针对外出的连接，对于进来的连接由tcp_retries1控制
默认值: 6
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

- tcp_tw_recycle
意义:是否打开快速 TIME-WAIT sockets 回收
默认值: 0
相关tcp/ip原理:

- tcp_tw_reuse
意义:是否允许重新应用处于TIME-WAIT状态的socket用于新的TCP连接
默认值: 0
相关tcp/ip原理:

- tcp_window_scaling
意义:设置tcp/ip会话的滑动窗口大小是否可变
默认值: 1
相关tcp/ip原理:

- tcp_wmem
意义:三个值，分别是
> Min：为TCP socket预留用于发送缓冲的内存最小值。每个tcp socket都可以在建立后使用它。
> Default：为TCP socket预留用于发送缓冲的内存数量，默认情况下该值会影响其它协议使用的net.core.wmem_default 值，一般要低于net.core.wmem_default的值。
> Max：用于TCP socket发送缓冲的内存最大值。该值不会影响net.core.wmem_max，"静态"选择参数SO_SNDBUF则不受该值影响

默认值: 4096	16384	4194304
相关tcp/ip原理:

- tcp_workaround_signed_windows
意义:0：假定远程连接端正常发送了窗口收缩选项，即使对端没有发送.
1：假定远程连接端有错误,没有发送相关的窗口缩放选项
默认值: 0
相关tcp/ip原理:

- udp_mem
意义:三个值，分别是
> low：当UDP使用了低于该值的内存页面数时，UDP不会考虑释放内存。
> presure：当UDP使用了超过该值的内存页面数量时，UDP试图稳定其内存使用，进入pressure模式，当内存消耗低于low值时则退出pressure状态。
> high：允许所有UDP sockets用于排队缓冲数据报的页面量

默认值: 88902	118538	177804
相关tcp/ip原理:

- udp_rmem_min
意义:
默认值: 4096
相关tcp/ip原理:

- udp_wmem_min
意义:
默认值: 4096
相关tcp/ip原理:

- xfrm4_gc_thresh
意义:
默认值: 2147483647
相关tcp/ip原理:

### 2.cipso缓存配置
- cipso_cache_bucket_size
意义:限制cipso缓存项的数量，如果在缓存中新添加一行超出了这个限制，那么最旧的缓存项会被丢弃以释放出空间
默认值: 10
相关tcp/ip原理:

- cipso_cache_enable
意义:是否启用cipso缓存
默认值: 1
相关tcp/ip原理:

- cipso_rbm_optfmt
意义:是否开启cipso标志优化选项
默认值: 0
相关tcp/ip原理:

- cipso_rbm_strictvalid
意义:是否开启cipso选项的严格检测
默认值: 1
相关tcp/ip原理:


- fib_multipath_use_neigh
意义:
默认值: 0
相关tcp/ip原理:

- fwmark_reflect
意义:
默认值: 0
相关tcp/ip原理:

### 3. icmp相关配置
- icmp_echo_ignore_all
意义:
默认值: 0
相关tcp/ip原理:

- icmp_echo_ignore_broadcasts
意义:忽略所有接收到的icmp echo请求的广播
默认值: 1
相关tcp/ip原理:

- icmp_errors_use_inbound_ifaddr
意义:
默认值: 0
相关tcp/ip原理:

- icmp_ignore_bogus_error_responses
意义:
默认值: 1
相关tcp/ip原理:

- icmp_msgs_burst
意义:
默认值: 50
相关tcp/ip原理:

- icmp_msgs_per_sec
意义:
默认值: 1000
相关tcp/ip原理:

- icmp_ratelimit
意义:
默认值: 1000
相关tcp/ip原理:

- icmp_ratemask
意义:
默认值: 6168
相关tcp/ip原理:


### 4. igmp相关配置
- igmp_link_local_mcast_reports
意义:
默认值: 1
相关tcp/ip原理:

- igmp_max_memberships
意义:
默认值: 20
相关tcp/ip原理:

- igmp_max_msf
意义:
默认值: 10
相关tcp/ip原理:

- igmp_qrv
意义:
默认值: 2
相关tcp/ip原理:

### 5. ip层配置
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

#### ip数据报重组相关
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
