---
layout: default
title: redis pipeline��ʵ�ַ�ʽ
---
##pipeline
redis��pipeline(�ܵ�)��������������û�У���redis��֧��pipeline�ģ������ڸ������԰��client�ж�����Ӧ��ʵ�֡� 
֮����������û������ΪRedis������һ��csģʽ��tcp server, client����ͨ��һ��socket����������������� 
ÿ�������������clientͨ�����������ȴ�redis����˴���redis����˴�����󽫽�����ظ�client�� 
����ͨ��pipeline��ʽ��client������һ�𷢳���redis server�ᴦ�����������󣬽����һ��������client,�Ӷ���ʡ�����������ӳٿ����� redis��֧������pipeline��ʽ.
##redis-lua
redis-lua��redis��lua�ͻ��ˣ�ͨ��һ�ν������ӣ�Ȼ����в���������������Դ�� redis-lua��pipeline��ʵ�ַ�ʽ�ǣ�
local replies = client:pipeline(function(p)
p:incrby('counter', 10)
p:incrby('counter', 30)
p:get('counter')
end)    
��Һã������ص㡣
nginx��lua-resty-redis
�����nginx��һ��ģ�飬����Ĵ󲿷�ָ���redis-lua��һ���ģ�Ӧ���Ƕ�redis-lua��һЩ���졣 
���������pipeline��ʵ�ַ�ʽ��Щ��ͬ�������ʵ�ַ�ʽ�ǣ�
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
����д�����ʱ���ҷ������ߵĲ�ͬ�Ļ�������Ϊredis-lua�����õ���һ���������Ұ�redis-lua�Ĵ���ŵ�nginx.conf��ʱ���ֱ���
 attempt to call method 'pipeline' (a nil value)
    stack traceback:    
��������ǣ���ȻͬΪlua�ܽ������������Ƿ���nginx����ģ�i/o���Ǻ�nginx��i/o�йأ�ֱ�ӷŽ�ȥ�ǲ��ܹ������ġ�over��

