---
layout: post
title: nginx how to support chinese
---

-1,firstly ,promise the linux lang is UTF-8

env|grep LANG

LANG=en_US.UTF-8

=============================

-2,nginx.conf seting utf-8

server

{

listen 80;

server_name .inginx.com ;

index index.html index.htm index.php;

root /usr/local/nginx/html/inginx.com;

charset utf-8;

}
=========================

disappiontment



