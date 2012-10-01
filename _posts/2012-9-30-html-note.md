---
layout: default
title: html 学习笔记
---
虽然很多语言都是可以转换为html的标准格式，但是很多操作都是需要对html格式进行直接操作，所以基本格式还是要了解的。
###学习资料：[十步学会css建站](http://jorux.com/archives/10steps-built-web-with-css/#c1)
本来想笔记要记得详细点的，后来实在是写不下了，就简单记下关键字吧，到时候看下唤醒下沉睡的记忆，不会再去查。嗯，就这样。  
> 
    <head>  头
    <title> 显示在标题栏
    <body> 内容
    <a href="http://www.w3school.com.cn">This is a link</a> 链接
    <meta http-equiv="Content-type" content="text/html; charset=UTF-8" /> 给爬虫看的
    <meta http-equiv=″refresh″ content=″2; URL=http://www.root.net″〉 2s后刷新页面

    <meta http-equiv="Page-Enter" content="revealTrans(duration=5.0, transition=20)"> 页面跳转的特效
    <meta http-equiv="Page-Exit" content="revealTrans(duration=5.0, transition=20)"> 这个也是


>

##css
>
    <div id="page-container"> :DIV元素是用来为HTML文档内大块（block-level）的内容提供结构和背景的元素
    Hello world.
    </div>
    上面是写在html里面的，下面是写在css里面的。
    #page-container {
    width: 760px;
    background: red;
    }
>
