---
layout: default
title: linux下安装lua常见或不常见问题(system：centos)
---
## 里面充满了大量的link和参考文章，慎入！！┐（─__─）┌  
首先当然是下载[lua 5.2](http://www.lua.org/download.html)  
接下来就是对照[操作手册](http://www.lua.org/manual/5.2/readme.html)安装了。    
如果你不幸跟我一样出现这个：  
>
    luaconf.h:275:31: error: readline/readline.h: No such file or directory  
    luaconf.h:276:30: error: readline/history.h: No such file or directory  
>

那么是因为你没有安装readline包，那么如果你能联网，那么你最好就：  
> 
        yum install readline-devel.x86_64  

不然的话呢，就和我一样去：  
http://www.bioinf.org.uk/software/profit/doc/node17.html  
下载一个安装包，并安装文档进行安装~╮(╯▽╰)╭
然后你make install 后，回到lua，在make install的时候，  
终于可以安装了~欧也~  
但是你现在正为此兴高采烈的话，我只能说你还是 **图样图森普了!!!** 
你在命令行打lua的时候，你会发现：  
>
        error while loading shared libraries: libpq.so.5: cannot open shared object file: No such file or directory  

你看一用ldd 查看/usr/local/bin里面的lua：  
> 
        ldd /usr/local/bin/lua  

会发现：libreadline.so.6 =>no found  
这是因为你的动态函数库没有加载到高速缓存。  
这时候你需要配置vi /etc/ld.so.conf  
加入 bin的路径（注意这里加的是路径哟。直接在第二次添上路径就可以了，不要加在第一行后面）  
如[这篇文章](http://hi.baidu.com/sing520/item/d27bbd39cb2d72fe97f88d5b)所言，  
再运行lua,就可以了。  
当然，你可能还会遇到一些其它的奇奇怪怪的问题，例如：[像什么要configure神马的](http://storysky.blog.51cto.com/628458/345982)    
好了,这时候你差不多就可以用基本的lua了。

但是因为lua是没有位运算的，所以我们有时候需要一些补丁啊什么的来补充一下。这个也是蛮麻烦的。  
本来是下了个[bitop](http://bitop.luajit.org/)但是这个不知道怎么安装，看了说明还是没看懂。
在embedding lua bitop的第二步时候不知道应该怎么弄，是要在Makefile里面引用另外一个makefile的意思么？惭愧的是我对shell不熟，所以就算了。
然后改用https://github.com/davidm/lua-bit-numberlua ,这个设置路径后就调用下就可以了。
调用的时候在lua的文件写下这两句：  
>
    package.path = '/home/test/lua-bit-numberlua/lmod/?.lua;' .. package.path  
    local bit = require 'bit.numberlua'

这个是我在看了lua-bit-number的[test.lua](https://github.com/davidm/lua-bit-numberlua/blob/master/test.lua)才知道的。%>_<%~ 
这个故事告诉我们 **每一个新知识点要有一个实例是多么多么的重要啊！！** 

这时候你的lua就能实现位运算了。是么是么？ **不要告诉我有问题啊！！！！！要是有我也不知道了~摔！！　（╯‵□′）╯︵┴─┴**

好了，到这里你就能在这里能实现一些基本的位运算了。但是呢，处理位运算的觉得这不是lua擅长的东西。所以有时候我们就需要调用C的函数，  
这时候我们就可以开始学习如何用C去写一个函数给lua调用了。这安装编译也是件麻烦的事，所以接下来我们来讲下C API,由于接下来的内容已经不是
这篇文章的内容了，所以需要另开一篇，欲知后事如何，且听下章分解。

●ω●

