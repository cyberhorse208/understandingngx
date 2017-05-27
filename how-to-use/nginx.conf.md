# nginx.conf 详细说明

本文将试图把nginx.conf中能够提供的配置指令全部作一个完整的说明（非nginx源码包含的模块提供的指令除外）


## nginx.conf 文件结构
```bash
#main全局相关配置

#用户
#log配置
#进程相关配置

#事件模型相关配置
events
{
 ...
}

#http相关配置，nginx.conf的核心配置
http
{
	#http相关全局配置
	
	#负载均衡配置，如上游服务器列表，访问、缓存策略等
	upstream xxx 
	{
		server xxxxx
	}
	
	#虚拟主机配置，可以有多个server块，表示有多个主机
	server 
	{
		#虚拟主机全局配置，如监听ip，端口，主机名字等
		
		#url特征对应处理方法配置 
		location url特征 {
			#处理方法
		}
	}
	
	server
	{
	}
}
```

## nginx.conf 详细配置指令说明
参考 www.cnblogs.com/hunttown/p/5759959.html
```bash
#指令功能：定义Nginx运行的用户和用户组
#指令格式： user 用户名 [用户组]；
#参考指令
user nobody;

#指令功能：配置nginx进程数，建议设置为等于CPU总核心数。
#指令格式： worker_processes n;
#参考指令
worker_processes 1;
 
 #指令功能：设置cpu亲缘性，就是绑定worker到指定cpu上，使得worker不需要在多个cpu间进行切换，从而避免切换cpu带来的上下文恢复、调度开销，提高性能。
#指令格式： worker_cpu_affinity mask
#参考指令（四核）
worker_cpu_affinity 0001 0010 0100 1000;
 
 
#指令功能：全局错误日志配置
#指令格式：error_log logfile [ debug | info | notice | warn | error | crit ]
#参考指令
error_log logs/error.log info;

#指令功能：指定nginx进程pid文件
#指令格式：pid path-to-pid
#参考指令
pid /usr/local/nginx/logs/nginx.pid;

#指令功能：指定进程可以打开的最大文件描述符数目（nginx打开的一个文件描述符一般都是对应一个网络连接）。理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
#现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
#这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
#指令格式：worker_rlimit_nofile n;
#参考配置
worker_rlimit_nofile 65535;

events 
{
	#指令功能：设置事件模型
	#指令格式： use [epoll|kqueue|dev/poll|select|poll]
	#参考指令 （linux上）
	use epoll;
	
	#指令功能：配置单个进程最大连接数（最大连接数=连接数*进程数）
    			#根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。
    	#指令格式： worker_connections n;
    	#参考指令：
    	worker_connections 65535;

    	#指令功能：设置tcp连接keepalive超时时间，单位为秒；超过此时间，连接上还没有新的数据包到来，则关闭此连接。
    	#指令格式：keepalive_timeout n;
    	#参考指令：
    	keepalive_timeout 60;

    	#指令功能：设置客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。但也有client_header_buffer_size超过4k的情况，但是client_header_buffer_size该值必须设置为“系统分页大小”的整倍数。
    	#指令格式：client_header_buffer_size n;
    	#参考指令：
    	client_header_buffer_size 4k;

    	#指令功能：为打开文件指定缓存，默认不启用。
    	#指令格式：open_file_cache max=n inactive=time;
    		max指定缓存文件数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    	#参考指令：
    	#open_file_cache max=65535 inactive=60s;

    	#指令功能：配置指多长时间检查一次缓存的有效信息。
    	#指令格式：open_file_cache_valid time;
    		time默认值60s;
    	#参考指令：
    	#open_file_cache_valid 60s;

    	#指令功能：指定open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果inactive时间内使用次数超过这个数字，文件描述符一直是在缓存中打开的；如果没有超过，它将被移除缓存。
    	#指令格式：open_file_cache_min_uses number ;
    		默认值：open_file_cache_min_uses 1;
    	#参考指令：
    	#open_file_cache_min_uses 1;
    
    	#指令功能：指定是否在搜索一个文件时记录cache错误.
    	#指令格式：open_file_cache_errors on | off 
    		默认值:open_file_cache_errors off 
    	#参考指令：
    	#open_file_cache_errors off;
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http
{
    #文件扩展名与文件类型映射表
    include mime.types;

    #默认文件类型
    default_type application/octet-stream;

    #默认编码
    #charset utf-8;

    #服务器名字的hash表大小
    #保存服务器名字的hash表是由指令server_names_hash_max_size 和server_names_hash_bucket_size所控制的。参数hash bucket size总是等于hash表的大小，并且是一路处理器缓存大小的倍数。在减少了在内存中的存取次数后，使在处理器中加速查找hash表键值成为可能。如果hash bucket size等于一路处理器缓存的大小，那么在查找键的时候，最坏的情况下在内存中查找的次数为2。第一次是确定存储单元的地址，第二次是在存储单元中查找键 值。因此，如果Nginx给出需要增大hash max size 或 hash bucket size的提示，那么首要的是增大前一个参数的大小.
    server_names_hash_bucket_size 128;

    #客户端请求头部的缓冲区大小。这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    client_header_buffer_size 32k;

    #客户请求头缓冲大小。nginx默认会用client_header_buffer_size这个buffer来读取header值，如果header过大，它会使用large_client_header_buffers来读取。
    large_client_header_buffers 4 64k;

    #设定通过nginx上传文件的大小
    client_max_body_size 8m;

    #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    #sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。
    sendfile on;

    #开启目录列表访问，合适下载服务器，默认关闭。
    #autoindex on;

    #此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    tcp_nopush on;
     
    tcp_nodelay on;

    #长连接超时时间，单位是秒
    keepalive_timeout 120;

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k;    #最小压缩文件大小
    gzip_buffers 4 16k;    #压缩缓冲区
    gzip_http_version 1.0;    #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2;    #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;    #压缩类型，默认就已经包含textml，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;

    #开启限制IP连接数的时候需要使用
    #limit_zone crawler $binary_remote_addr 10m;



    #负载均衡配置
    #nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
    upstream backend {
     	#指令功能：设置上游服务器访问规则
     	#指令格式：server [ip|servername]:port [weight=n] [down] [backup] [max_fails=n fail_timeout=time]  
     	        #down:表示单前的server暂时不参与负载
        	#weight:weight越大，负载的权重就越大。
        	#max_fails：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream模块定义的错误
        	#fail_timeout:max_fails次失败后，暂停的时间。
        	#backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。
     	#参考指令：
        #server 192.168.80.121:80 weight=3;
        #server 192.168.80.122:80 weight=2;
        #server 192.168.80.123:80 weight=3;
        #server 192.168.80.124:80 weight=4 down;
        #server 192.168.80.125:80 backup;
	#server 192.168.80.126:80 max_fails=2 fail_timeout=10s weight=2;
	
	#指令功能：设置挑选上游服务器的方法
	#指令格式：[ip_hash | fair | hash_method algo]
        	#1、轮询（默认）
        		#每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
        		#指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
        	#2、ip_hash
        		#每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
        	#3、fair（第三方）
        		#按后端服务器的响应时间来分配请求，响应时间短的优先分配。
        	#4、url_hash（第三方）
        		#按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
        		#例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
	#参考指令：
	#ip_hash;
        
    }
     
     
     
    #虚拟主机的配置
    server
    {
        #监听端口
        listen 80;

        #域名可以有多个，用空格隔开
        server_name www.sample.com sample.com;
        index index.html index.htm index.php;
        root /data/www/jd;

        #对******进行负载均衡
        location ~ .*.(php|php5)?$
        {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }
         

        #日志格式设定
        #$remote_addr与$http_x_forwarded_for用以记录客户端的ip地址；
        #$remote_user：用来记录客户端用户名称；
        #$time_local： 用来记录访问时间与时区；
        #$request： 用来记录请求的url与http协议；
        #$status： 用来记录请求状态；成功是200，
        #$body_bytes_sent ：记录发送给客户端文件主体内容大小；
        #$http_referer：用来记录从那个页面链接访问过来的；
        #$http_user_agent：记录客户浏览器的相关信息；
        #通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" '
        '"$http_user_agent" $http_x_forwarded_for';
         
        #定义本虚拟主机的访问日志
        access_log  /usr/local/nginx/logs/host.access.log  main;
        access_log  /usr/local/nginx/logs/host.access.404.log  log404;
         
        #对 "/" 启用反向代理
        location / {
            proxy_pass http://backend/;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
             
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             
            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;

            #允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            #缓冲区代理缓冲用户端请求的最大字节数，
            #如果把它设置为比较大的数值，例如256k，那么，无论使用firefox还是IE浏览器，来提交任意小于256k的图片，都很正常。如果注释该指令，使用默认的client_body_buffer_size设置，也就是操作系统页面大小的两倍，8k或者16k，问题就出现了。
            #无论使用firefox4.0还是IE8.0，提交一个比较大，200k左右的图片，都返回500 Internal Server Error错误
            client_body_buffer_size 128k;

            #表示使nginx阻止HTTP应答代码为400或者更高的应答。
            proxy_intercept_errors on;

            #后端服务器连接的超时时间_发起握手等候响应超时时间
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout 90;

            #后端服务器数据回传时间(代理发送超时)
            #后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
            proxy_send_timeout 90;

            #连接成功后，后端服务器响应时间(代理接收超时)
            #连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
            proxy_read_timeout 90;

            #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            #设置从被代理服务器读取的第一部分应答的缓冲区大小，通常情况下这部分应答中包含一个小的应答头，默认情况下这个值的大小为指令proxy_buffers中指定的一个缓冲区的大小，不过可以将其设置为更小
            proxy_buffer_size 4k;

            #proxy_buffers缓冲区，网页平均在32k以下的设置
            #设置用于读取应答（来自被代理服务器）的缓冲区数目和大小，默认情况也为分页大小，根据操作系统的不同可能是4k或者8k
            proxy_buffers 4 32k;

            #高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;

            #设置在写入proxy_temp_path时数据的大小，预防一个工作进程在传递文件时阻塞太长
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
            proxy_temp_file_write_size 64k;
        }
         
         
        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status on;
            access_log on;
            auth_basic "NginxStatus";
            auth_basic_user_file confpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生。
        }
         
        #本地动静分离反向代理配置
        #所有jsp的页面均交由上游服务器处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://backend;
        }
         
        #所有静态文件由nginx直接读取不经过tomcat或resin
        location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|
        pdf|xls|mp3|wma)$
        {
            expires 15d; 
        }
         
        location ~ .*.(js|css)?$
        {
            expires 1h;
        }
    }
```