---
layout: default
title: nginx限制IP连接数的范例参考
---
nginx 限制ip并发数，nginx限制IP连接数的范例参考：




如何Nginx限制同一个ip的连接数，限制并发数目:  


1.添加limit_zone和limit_req_zone 
这个变量只能在http使用 :

    vi /export/servers/nginx/conf/nginx.conf 
    limit_zone one  $binary_remote_addr  20m;
    limit_req_zone  $binary_remote_addr  zone=req_one:20m rate=12r/s;


2.添加limit_conn 和limit_req
这个变量可以在http, server, location使用 
我是限制nginx上的所有服务，所以添加到http里面 
（如果你需要限制部分服务，可在nginx/conf/domains里面选择相应的server或者location添加上便可）


    vi /export/servers/nginx/conf/nginx.conf 
    
    limit_zone one $binary_remote_addr 20m;
    limit_req_zone $binary_remote_addr zone=req_one:20m rate=12r/s;
    limit_conn one 10;
    limit_req zone=req_one burst=120;

参数详解(数值按具体需要和服务器承载能力设置,):  

    limit_zone，是针对每个变量(这里指IP，即$binary_remote_addr)定义一个存储session状态的容器。这个示例中定义了一个20m的容器，按照32bytes/session，可以处理640000个session。
    limit_req_zone 与limit_zone类似。rate是请求频率. 每秒允许12个请求。
    limit_conn  one 10 : 表示一个IP能发起10个并发连接数
    limit_req: 与limit_req_zone对应。burst表示缓存住的请求数。

范例:

    http
    {
    limit_zone one  $binary_remote_addr  20m;
    limit_req_zone  $binary_remote_addr  zone=req_one:20m rate=12r/s;
    
    limit_conn   one  10;
    limit_req   zone=req_one burst=120;
    
    
    server  {
            listen          80;
            server_name     status.xxx.com ;
    
            location / {
                     stub_status            on;
                     access_log             off;
            }
    
    }
    }





 
3.重启nginx 

  /export/servers/nginx/sbin/nginx -s reload


Nginx限制流量/限制带宽
具体参考官方文档

[关于limit_zone](http://wiki.nginx.org/HttpLimitZoneModuleChs)  
[关于limit_req](http://wiki.nginx.org/HttpLimitReqModule)

##nginx白名单设置##
  以上配置会对所有的ip都进行限制，有些时候我们不希望对搜索引擎的蜘蛛或者某些自己的代理机过来的请求进行限制，
对于特定的白名单ip我们可以借助geo指令实现。

先在nginx的请求日志进行统计，查看那个ip的访问量比较大，
运行:

     cat access.log | grep "03/Jun" |awk '{print $1}'|sort |uniq -c|sort -nrk 1|head -n 10
     #列出访问日志里面在6月3号这天前10个访问量最大的ip.

接下来就可以对这些IP进行分析了。看哪些需要进行白名单设置。

     http{
         geo  $limited  { # the variable created is $limited
          default          1;
          127.0.0.1/32     0;
          10.12.212.63     0;
        }
        
        map $limited $limit {
        1 $binary_remote_addr;
        0 "";
        }
        
        limit_zone one  $binary_remote_addr  20m;
        limit_req_zone  $limit  zone=req_one:20m rate=20r/s;
        
        limit_conn   one  10;
        limit_req   zone=req_one burst=120;
    }
    
上面两个需要用到map和geo模块，这是nginx自带的模块，有的运维喜欢把他们关闭，自己./sbin/nginx -V 留意一下。把配置的--whithout-XXX-module 去掉重新编译一下就可以了。
上面这段配置的意思是：

1.geo指令定义了一个白名单$limited变量，默认值为1，如果客户端ip在上面的范围内，$limited的值为0

2.使用map指令映射搜索引擎客户端的ip为空串，如果不是搜索引擎就显示本身真实的ip，这样搜索引擎ip就不能存到limit_req_zone内存session中，所以不会限制搜索引擎的ip访问



PS:
##获取客户端的真实IP##
顺带一提，为了获取客户端的真实IP。该模块需要安装read_ip模块，运维应该默认有安装。没有的话也可自行安装：
配置方式相当简单，重新编译 Nginx 加上 --with-http_realip_module 参数，如：

    ./configure --prefix=/opt/nginx --with-http_stub_status_module  --with-pcre=../pcre-6.6 --with-http_realip_module
    make
    make install

在server中增加:

    set_real_ip_from   192.168.1.0/24;
    set_real_ip_from   192.168.2.1;
    real_ip_header     [X-Real-IP|X-Forwarded-For];
需要说明的地方就是设置IP源的时候可以设置单个IP，也可以设置IP段，另外是使用X-Real-IP还是X-Forwarded-For，取决于前面的服务器有哪个头。

set_real_ip_from 设置的IP端可以让运维查看日志，看下你的请求是来自哪些ip段。

重新加载一下服务，差不多就OK了。

再查看日志的话，应该可以看到客户端的真实IP了。


注意：如果未安装该模块的话你的获取到的IP端可能是来自前端代理（如squid）的IP，结果就是多个用户被当成单个用户对待，导致应用不能响应。
参考:http://hi.baidu.com/thinkinginlamp/item/e2cf05263eb4d18e6e2cc3e6


再PS一下：
自测:
有条件的自己可以用ab或者webben自测一下。

未安装前压测的话，因为有大量请求，所以access.log会有大量日志，而error.log日志没有变化。

    [root@qrwefsdf talk]# webbench  -c 30 -t 30 http://xxx.com  
    Webbench - Simple Web Benchmark 1.5
    Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
    Benchmarking: GET http://xxx.com  
    30 clients, running 30 sec.
    Speed=193468 pages/min, 1254317 bytes/sec.
    Requests: 96734 susceed, 0 failed.

安装后会发现很多超出的请求会返回503,所以access.log日志变化不快，error.log有大量记录,提示limit_reque缓住了多少请求。

    [root@qrwefsdf talk]# webbench  -c 30 -t 30 http://xxxx.com
    Webbench - Simple Web Benchmark 1.5
    Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
    Benchmarking: GET http://xxx.com  
    30 clients, running 30 sec.
    Speed=120 pages/min, 778 bytes/sec.
    Requests: 60 susceed, 0 failed.


