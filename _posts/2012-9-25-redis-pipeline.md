---
layout: default
title: redis pipeline的实现
---
##Pipeline
redis的pipeline(管道)功能在命令行中没有，但redis是支持pipeline的，而且在各个语言版的client中都有相应的实现。    
之所以命令行没有是因为Redis本身是一个cs模式的tcp server, client可以通过一个socket连续发起多个请求命令。     
每个请求命令发出后client通常会阻塞并等待redis服务端处理，redis服务端处理完后将结果返回给client。    
所以通过pipeline方式将client端命令一起发出，redis server会处理完多条命令后，将结果一起打包返回client,从而节省大量的网络延迟开销。 redis是支持这种pipeline方式.      
##[redis-lua](https://github.com/nrk/redis-lua)
redis-lua是redis的lua客户端，通过一次建立连接，然后进行操作。具体规则参照源码 redis-lua的pipeline的实现方式是： 
        local replies = client:pipeline(function(p)
        p:incrby('counter', 10)
        p:incrby('counter', 30)
        p:get('counter')
        end)    
###大家好，下面是重点君。
##nginx的[lua-resty-redis](https://github.com/agentzh/lua-resty-redis)            
这个是nginx的一个模块，里面的大部分指令和redis-lua是一样的，应该是对redis-lua的一些改造。          
但是里面的pipeline的实现方式有些不同，这边是实现方式是：    
        red:init_pipeline()
        red:incrby('counter', 10)
        red:incrby('counter', 30)
        red:get('counter')
        local repies,err = red:commit_pipeline()
        if not repies then
                ngx.say("erro",err)
                return
        end
        for i,res in ipairs(repies) do
        ngx.say("counter:",res)
        end   
当我写到这的时候，我发现两者的不同的还有是因为redis-lua里面用的是一个函数，我把redis-lua的代码放到nginx.conf的时候发现报错
         attempt to call method 'pipeline' (a nil value)
            stack traceback:    
个人理解是：虽然同为lua能解析，但是它是放在nginx里面的，i/o还是和nginx的i/o有关，直接放进去是不能够解析的。over。


