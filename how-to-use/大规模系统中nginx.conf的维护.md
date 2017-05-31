# 大规模系统中nginx.conf的维护

> 在一般规模网站系统中，可能也就几台nginx设备，此条件下对nginx进行配置维护相对比较简单，可以直接使用一个nginx.conf进行控制即可。但是，在大规模网站系统，有异地、多机房、几十台nginx设备时，再使用简单的nginx.conf进行控制可能就会带来各种问题。比如配置不一致、更新不同步之类，造成访问异常。
> 本文主要参考 [张开涛](http://www.jinnianshilongnian.iteye.com/blog/2339876 ) 的博客和书 ，将大规模系统中nginx.conf的维护方法做一个笔记。

