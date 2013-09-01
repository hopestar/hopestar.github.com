---
layout: default
title:  nginx对redis取数据的不同方式
---
#1，原生态方式
用的是[httpredis2module](http://wiki.nginx.org/HttpRedis2Module)的原始方式。    
 新建一个upstream：   
     upstream redis_server{
            server 127.0.0.1:6379 weight=1;
    }
    
    location /foo {
        set $value 'first';
        redis2_query set one $value;
        redis2_query get one;
        redis2_query set one two;
        redis2_query get one;
        redis2_pass redis_server;
    }
这里用了[管道](http://hopestar.github.com/2012/09/25/redis-pipeline/)，将几条命令一起发过去。   
这里没有用到lua,所以不能实现逻辑，返回的是redis的原始数据。

#2，普通方式
在nginx.conf里面新建一个location 
    
        location = /redis2 {
        internal;
        redis2_raw_queries $args $echo_request_body;
        redis2_pass redis_server ;
         }

然后就是插入lua脚本，

    local parser = require "redis.parser"
    ngx.req.read_body()
    local args = ngx.req.get_post_args()
    local key=ngx.var.arg_key;
    local value=nil;
        local reqs = {
            {"get", key}
        }
    
        local raw_reqs = {}
        for i, req in ipairs(reqs) do
            table.insert(raw_reqs, parser.build_query(req))
        end
    
        local res = ngx.location.capture("/redis2?" .. #reqs,
            { body = table.concat(raw_reqs, "") })
        if res.status ~= 200 or not res.body then
            ngx.log(ngx.ERR, "failed to query redis")
            ngx.exit(500)
        end
    
       local replies = parser.parse_replies(res.body, #reqs)
        for i, reply in ipairs(replies) do
            if(reply[1] ~= nil) then
                    ngx.say(reply[1])
            end
        end     
用的是ngx.location.capture调用redis2进行过渡（是这意思吧？是因为lua里面没有proxy_pass的缘故么？）。

#3 在lua内部的连接方式    
用[lua-resty-redis](https://github.com/agentzh/lua-resty-redis)的方式，相对于前面几种方式的好处是你不是用proxy_pass，
所以你可以对连接（redis:new(),redis:connect()）等方式写在一起，而且对连接的状态（如一次连接的操作次数get_reused_times）
进行获取。不足就是目前还不知道要对集群的话怎么弄。应该可以自己写一个random函数对几个backend进行轮询了。不过感觉这样就没什么意思了。
local redis = require "resty.redis"
            local red = redis:new()

            red:set_timeout(1000) -- 1 sec

            -- or connect to a unix domain socket file listened
            -- by a redis server:
            --     local ok, err = red:connect("unix:/path/to/redis.sock")

            local ok, err = red:connect("127.0.0.1", 6379)
            if not ok then
                ngx.say("failed to connect: ", err)
                return
            end

            ok, err = red:set("dog", "an aniaml")
            if not ok then
                ngx.say("failed to set dog: ", err)
                return
            end

            ngx.say("set result: ", res)

            local res, err = red:get("dog")
            if not res then
                ngx.say("failed to get dog: ", err)
                return
            end

            if res == ngx.null then
                ngx.say("dog not found.")
                return
            end

            ngx.say("dog: ", res)

            red:init_pipeline()
            red:set("cat", "Marry")
            red:set("horse", "Bob")
            red:get("cat")
            red:get("horse")
            local results, err = red:commit_pipeline()
            if not results then
                ngx.say("failed to commit the pipelined requests: ", err)
                return
            end

            for i, res in ipairs(results) do
                if type(res) == "table" then
                    if not res[1] then
                        ngx.say("failed to run command ", i, ": ", res[2])
                    else
                        -- process the table value
                    end
                else
                    -- process the scalar value
                end
            end

            -- put it into the connection pool of size 100,
            -- with 0 idle timeout
            local ok, err = red:set_keepalive(0, 100)
            if not ok then
                ngx.say("failed to set keepalive: ", err)
                return
            end

            -- or just close the connection right away:
            -- local ok, err = red:close()
            -- if not ok then
            --     ngx.say("failed to close: ", err)
            --     return
            -- end
            
