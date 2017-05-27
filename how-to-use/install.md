# ubuntu上nginx的安装

## 安装环境
- ubuntu 16.04 lts


## 安装步骤
1， 安装pcre
- 从 www.pcre.org 下载源码（注意下载pcre，不是pcre2） 
- 解压缩，进入解压缩目录
- ./configure
- make
- sudo make install

2， 安装nginx
- nginx.org/en/download.html 下载最新源码包。我选择了stable version的最新版本 nginx_1.12.0
- 解压，进入解压缩目录
- ./configure
- make
- sudo make install
	nginx 将安装到 /usr/local/nginx目录
