# how to make a nginx module

## 1.nginx模块概述
	nginx的基本架构是 事务处理框架 + 功能扩展模块 的形式。	
	nginx的事务处理框架将绝大部分与OS、网络打交道的代码都封装实现，然后将数据提供给各个功能模块，按照一定的顺序进行处理。
	各个功能模块无须考虑OS和网络的复杂性，可以专注于数据处理业务。	
	nginx功能模块也分为两类：核心模块和普通业务模块。	
	nginx的核心模块提供了其他模块需要使用的一些共性功能，比如：	
		ngx_core_module
		ngx_errlog_module
		ngx_conf_module
		ngx_regex_module
		ngx_events_module
		ngx_http_module	
	其他普通业务模块都是基于这些核心模块来提供各种自定义的功能。	
	nginx使用模块的方式有两种：	
		-- 静态装载，与nginx源码打包到一起，成为一个可执行文件。因此，给nginx添加新模块时，必须要替换nginx可执行文件。	
		-- 动态装载，编译成一个动态库，nginx在启动时根据nginx.conf文件的配置动态链接。此功能在nginx的1.9.11版本开始提供。		
	nginx官方提供了很多模块，包括所有核心模块和一些业务模块。
	核心模块和一部分业务模块默认被打包到nginx可执行文件中，另外还有一些模块需要用户自行决定是否使用。
	nginx用这种方式确保功能尽可能简介可靠，从而提高nginx处理能力。
	

## 2. 静态加载模块
- 静态加载官方模块
	在configure时，添加 --with-xxx 即可，比如
```bash
./configure --with-http_geoip_module \
　　--with-http_image_filter_module  \
　　--with-mail  \
　　--with-stream  \
　　--with-http_xslt_module 
```
- 静态加载第三方模块
	在configure时，添加--add-module=path_to_module_dir  即可，比如
```bash
	./configure --add-module=/home/will/nginx-1.12.0/work/modules/upstream
```	
	
## 3. 动态加载模块
采用动态加载的模块，编译完成后，生成对应模块名字.so 的文件在编译目录的objs目录下。
可以再make install将模块自动安装到 nginx的安装目录的子目录modules下。
- 动态加载官方模块
	在configure时，添加 --with-xxx=dynamic 即可，比如
```bash
./configure --with-http_geoip_module=dynamic \
　　--with-http_image_filter_module=dynamic \
　　--with-mail=dynamic \
　　--with-stream=dynamic \
　　--with-http_xslt_module=dynamic
```
- 动态加载第三方模块
	模块如果想要编译成动态链接库，那么**必须使用新版本的模块config文件**（1.9.11后推荐使用的config文件）.
	在configure时，添加 --add-dynamic-module=path_to_module_dir 即可，比如：
```bash
	./configure --add-dynamic-module=/home/will/nginx-1.12.0/work/modules/upstream
```
	
	


## 4. nginx module 的组成部分
- 模块源码

	模块功能实现代码
	
- config
	文件名为config，其中包含模块编译配置：
		模块名字
		模块代码所在目录
		依赖
		环境变量设置等
		
### 4.1 旧版本的模块配置文件config
```bash
ngx_addon_name=模块名
HTTP_MODULES="$HTTP_MODULES 模块名"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/模块实现.c"
#如果模块实现没有.h文件，那么可以没有下面这个配置
NGX_ADDON_DEPS="$NGX_ADDON_DEPS $ngx_addon_dir/模块实现.h"
```

### 4.2 新版本的模块配置文件config
```bash
ngx_addon_name=模块名

if test -n "$ngx_module_link"; then
    ngx_module_type=HTTP
    ngx_module_name=模块名
    ngx_module_srcs="$ngx_addon_dir/模块实现.c"
    ngx_module_incs=
    ngx_module_deps=
    ngx_module_libs=
    
    . auto/module
else
    HTTP_MODULES="$HTTP_MODULES ngx_http_response_module"
    NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/模块实现.c"
    #如果模块实现没有.h文件，那么可以没有下面这个配置
    NGX_ADDON_DEPS="$NGX_ADDON_DEPS $ngx_addon_dir/模块实现.h"
fi
```	

### 4.3 一个简单的模块组成文件样例
```bash
will@will-ThinkPad-T400:~/nginx-1.12.0/work/modules/upstream$ ll
total 28
drwxrwxr-x 2 will will  4096 5月  31 10:37 ./
drwxrwxr-x 6 will will  4096 5月  31 10:37 ../
-rw-rw-r-- 1 will will   175 5月  15 16:42 config
-rw-rw-r-- 1 will will 15123 5月  16 22:41 ngx_http_myupstream_module.c
will@will-ThinkPad-T400:~/nginx-1.12.0/work/modules/upstream$ cat config 
ngx_addon_name=ngx_http_myupstream_module
HTTP_MODULES="$HTTP_MODULES ngx_http_myupstream_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_myupstream_module.c"
will@will-ThinkPad-T400:~/nginx-1.12.0/work/modules/upstream$ 
```
		
## 5. 编写一个模块
### 5.1 http模块源码组织结构