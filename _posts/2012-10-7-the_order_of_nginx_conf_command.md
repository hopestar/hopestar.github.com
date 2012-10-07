---
layout: default
title: nginx配置指令的执行顺序
---
参考自agentzh的[sina blog](http://blog.sina.com.cn/openresty)
因为很多时候nginx的指令执行并不是按照指令在配置的位置由上到下执行的，而是按照nginx在加载
这些指令的时候的顺序来执行的。具体的在章亦春的blog里面写得很清楚，这里要写的只是按照nginx的启动顺序和
不同阶段对应的不同指令，以便书写配置的时候方便参考。    
nginx配置执行共有11个阶段，其中3个不支持注册处理程序，剩下有8个阶段可以注册处理程序。
>  
    
    ##nginx处理请求流程:
    
    1,post-read(在nginx读取解析请求头后执行): set_real_ip_from,read_ip_head;    
        
    2,server-rewrite(server中的rewrite):    
    
    3,find-config(负责location的跳转，不支持注册处理程序):
    
    4,rewrite：set族，rewrite_by_lua(末期)
    
    5,post-rewrite(内部跳转阶段，向find-config阶段回退，不支持注册处理程序)
    
    6,preaccess(预access):ngx_limint_req,ngx_limit_zone,set_real_ip_from等(该指令在post-read也注册了)
    
    7,access :allow,deny,access_by_lua(末期)
    
    8,post-access(不支持注册处理程序)；
    
    9,try-file:实习try-file功能，不支持注册；
    
    10,content(这里content handle是竞争的，一个location里面只能处理一个content handle，所以一个location里面
    这些命令中的只能出现一类):echo,echo_exec,proxy_pass,echo_location,content_by_lua;
    
    11,log:
    
    -----
    这里还有一个特殊阶段，因为这个阶段可以被很多阶段调用，所以比较特殊：
    output-filter:echo_before_body,echo_after_body;