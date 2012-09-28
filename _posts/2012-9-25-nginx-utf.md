---
layout: post
title: nginx 如何支持中文
---
在你用nginx对uri取参数的时候，nginx自带的set 和set_unescape_uri 都是默认不支持中文的，所以你在
对redis存取数据是不方便的。这就需要进行以下设置了。   
##1,firstly ,promise the linux lang is UTF-8    
    env|grep LANG
    LANG=en_US.UTF-8
    
##2,nginx.conf seting utf-8   
    server
    {
    listen 80;
    server_name .inginx.com ;
    index index.html index.htm index.php;
    root /usr/local/nginx/html/inginx.com;
    charset utf-8;
    }



