---
layout: default
title: 一致性哈希(Consistent Hash)不是很详细的详解
---
在文章的最后面我会贴两篇我参考的文章，写得很好，图也很漂亮，新手的话你可以先看下，然后不理解的话再来看我这篇，我是看了那两篇以后，
还有一点点不是很理解，然后问了下老大后才像明白的。嗯。好，开始了。  
（等等，刚才想了一下，还是强烈建议你们去看完那两篇后再来看我这个吧，感觉这些东西我全写出来就太繁琐了，我是个比较懒的人，
别人已经写好的东西我不想重复。╮(￣▽￣")╭ ...)

consistent hashing 是一种分布式系统中常用的算法。，单的说，在移除 / 添加一个 cache 时，它能够尽可能小的改变已存在 key 映射关系，尽可能的满足单调性的要求。
一个分布式的存储系统，要将数据存储到具体的节点上，如果采用普通的hash方法，将数据映射到具体的节点上，如key%N，key是数据的key，N是机器节点数，如果有一个机器加入或退出这个集群，则所有的数据映射都无效了，
如果是持久化存储则要做数据迁移，如果是分布式缓存，则其他缓存就失效了。
因此，引入了一致性哈希算法.    
#1,你需要知道的3个东西
##1.1,真实机器和虚拟节点的映射表(请允许我暂时称它为node表吧)
##1.2,一种hash规则
##1.3,根据object的哈希值得到server的哈希值的规则

废话少说，我先画张图吧。
![alt O(∩_∩)O~](/img/note/crazy.jpg "画图中....")  
好了，画完了。就是这样子了。    
![alt O(∩_∩)O~](/img/note/hash.png "hash")  

首先我们要明确一点，就是我们手头上是有两个东西（暂且让老衲称它为client和server吧）要做哈希的（不是一个哟！！）,哈希表的目的就是为了建立和client和server的联系。
![alt text][1]  
例如有1000个用户要访问我们的网站，我需要让10台服务器去处理。我希望根据每个用户（根据不同的pin值和信息做成一个key来判别）每次来的时候都
能访问相同的服务器，如果我们简单的用key%n=n,去得到一个数，然后根据这个数n去访问服务器n,那么这只是一层映射，如果你加减一个服务器，
key%(n+1),key%(n-1)都会令大部分用户访问的服务器改变了。所以我们需要的是一个中间层（呵呵哈，这是我自己提出来的名词），这个中间层
是一个数值空间(或者叫哈希空间，通常的 hash 算法都是将 value 映射到一个 32 为的 key 值，也即是 0~2^32-1 次方的数值空间；)，client映射到上面是一个数字，server映射到上面也是一个数字，然后，我们按照client的数字取比它大最少的而且有服务器对应的一个数字，
然后取出这个数字对应的服务器作为它要访问的服务器。（有点拗口，但是聪明如你，应该能懂的），这样一来，如果加减服务器的话，原来的client对应的
哈希值是没有变的，而原来没坏了的服务器因为机器名字没变，所以它的哈希值也没变，所以它们之间的对应是没有问题的。那么原来对应那台坏了的客户请求
就会迁移到另外一台服务器上。  
这时候问题来了。这又会造成一个“雪崩”的情况，即下一个节点由于承担了前个节点的数据，所以下一个节点的负载会变高，该节点很容易也宕机，这样依次下去，这样造成整个集群都挂了。
为此，引入了“虚拟节点”的概念：即把想象在这个环上有很多“虚拟节点”，数据的存储是沿着环的顺时针方向找一个虚拟节点，每个虚拟节点都会关联到一个真实节点，这就是
为什么我们需要一个node表的原因了。这张表就是维护真实节点和虚拟节点的映射关系。 将真实的节点虚拟成160倍（倍数越多越均匀）的虚拟节点
所以你看到的node表应该是一个个哈希后的数字和该数字对应的服务器的信息。例{"1234123"，ip=168.0.0.1}
这时候逻辑就是这样:  
![alt -_-!!!](/img/note/hash2.png "hash2")  
好了.接下来就是hash算法的选择。   
hash有很多种，像加法这些简单的碰撞率太高了，所以我们要挑种高效碰撞率又低的。现在一般是选择MD5，fvn,murmur等，
这有几种python实现的快速的算法[pyfasthash](http://code.google.com/p/pyfasthash/).你可以看下。  
我这里用的是murmurhash,因为据说比较快和碰撞率比较低，而且是我正在看nginx，nginx的ngx-http-split-clients模块也是用的murmurhash([可以参照源码](https://github.com/nginx/nginx/blob/master/src/core/ngx_murmurhash.c)).
murmurhash根据需求位数可分为很多种，这里提供了几种C/C++实现的版本[murmur_hash.cpp](https://github.com/jmhodges/murmur_hash/blob/master/ext/murmur/murmur_hash.cpp)，我用的就是里面的murmurhash64B。   

好了，又到了贴代码的时间了。我贴的是lua的代码。哈希方法用C做成一个动态链接库（怎么调用可参照本站lua与C交互这篇文章）放进来了。
>
    --load lib
    package.loadlib("/home/test/www/libhashlib.so", "luaopen_hashlib")()
    --real machine
    local shards = {
    {name="",ip="127.0.0.1",port=81,weight=1},
    {name="",ip="127.0.0.2",port=82,weight=1},
    {name="machine3",ip="127.0.0.3",port=83,weight=1},
    }
    
    
    --char to number base on ascii
    function chartonumber(char)
    local i=32
    local temp_char=char
     while i<126 do
     if temp_char == string.format('%c', i) then
     return i
     end
     i=i+1
     end
    end
    
    
    --table for keep virtual nodes and real nodes
    nodes={}
    
    --inital virtual nodes
    function virnode_init(shards,nodes)
    for i,shardinfo in pairs(shards) do
    -- print("i="..i,"shardinfo="..shardinfo.ip..":"..shardinfo.port,"weight:"..shardinfo.weight)
      if shardinfo.name==nil or shardinfo.name == "" then
            for n=0,160*shardinfo.weight do
            table.insert(nodes,{hashlib.murmurhash64b("SHARD-"..i.."-NODE-"..n),shardinfo})
            end
      else
            for n=0,160*shardinfo.weight do
            table.insert(nodes,{hashlib.murmurhash64b(shardinfo.name.."*"..shardinfo.weight..n),shardinfo})
            --i can't understand why add this for nodes.key,why not add shardinfo.name only
            end
      end
    end
    return
    end
    
    function getshardinfo(nodes,key)
    local i=0
    local nodeinfo=nil
    key=hashlib.murmurhash64b(key)
    print("key:"..key)
    table.sort(nodes,function(a,b) return a[1]<b[1] end)--nodes sort by nodes.name
      for i, nodeinfo in pairs(nodes) do
        if key<=nodeinfo[1] then
          return nodes[i]
        end
    --print(nodeinfo[1],key)
      end
      return nodes[1]
    end


测试用例自己写~我是不会给你写的~o(*￣▽￣*)o 



[1]: /img/note/example.jpg "example"



参考资料：
[一致性hash算法详解](http://blog.csdn.net/tianmo2010/article/details/6838312)  
[一致性哈希算法与Java实现](http://www.blogjava.net/hello-yun/archive/2012/10/10/389289.html)