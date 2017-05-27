# nginx 的使用

## nginx 安装目录说明
	nginx 默认安装到/usr/local/nginx目录下，此目录下的子目录功能为：
	- client_body_temp
		客户端发送过来的请求体，以文件的形式缓存在此目录
	- conf
		默认配置文件存放目录.一般我们只需要关注其中的nginx.conf文件。
	- fastcgi_temp
	- logs
		默认日志文件存放目录
	- proxy_temp
	- sbin
		nginx 可执行文件存放目录
	- scgi_temp
	- uwsgi_tem
	
	
## 运行nginx
- 将nginx可执行文件所在目录加入path环境变量中
	-- sudo echo "export PATH=$PATH:/usr/local/nginx/sbin/" >> /etc/profile
	-- source /etc/profile
- 启动nginx
	-- sudo nginx -c path_to_nginx.conf
	可以直接使用nginx/conf目录下的配置文件，也可以拷贝nginx.conf和mine.types到指定目录，然后使用此目录下的confi文件运行
- 停止nginx
	-- sudo nginx -s stop
- 更新配置文件
	-- sudo nginx -s reload
	
- 检查安装
	如果使用默认的nginx.conf，安装完毕后，在浏览器里输入 localhost， 如果显示nginx的欢迎界面，就表示一切顺利，成功安装了。
	
- 更新nginx程序
	-- 获得正在运行的nginx master的进程号n
		ps uax|grep nginx|grep master
	-- kill -WINCH n   //发送WINCH信号到旧的nginx主进程以杀掉旧的nginx子进程
	-- kill -QUIT n  // 退出旧的nginx主进程 
	-- rm -rf /usr/local/nginx/sbin/nginx.old  //删除老版本
	--现在新版本的nginx就已经替换老的nginx了，而且实现了不停机升级。
	
