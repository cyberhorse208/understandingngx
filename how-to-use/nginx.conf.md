# nginx.conf 详细说明

本文将试图把nginx.conf中能够提供的配置指令全部作一个完整的说明（非nginx源码包含的模块提供的指令除外）


## nginx.conf 文件结构
```bash
#全局相关配置

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
```