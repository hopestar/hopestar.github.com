---
layout: default
title: C和lua交互的简单例子
---
咳咳，我理解的事实应该是这样的。C和lua交互，有两种方式，第一种，C作为应用程序语言，Lua作为一个库使用；第二种，反过来，Lua作为程序语言，C作为库使用。
**这两种方式，C语言都使用相同的API与Lua通信，因此C和Lua交互这部分称为C API。**
**在C和Lua之间通信关键内容在于一个虚拟的栈。几乎所有的API调用都是对栈上的值进行操作，所有C与Lua之间的数据交换也都通过这个栈来完成。** 另外，你也可以使用栈来保存临时变量。栈的使用解决了C和Lua之间两个不协调
的问题：第一，Lua会自动进行垃圾收集，而C要求显示的分配存储单元，两者引起的矛盾。第二，Lua中的动态类型和C中的静态类型不一致引起的混乱。

具体参考[Programming in Lua](http://www.lua.org/pil/24.html)

首先我们看下这个仁兄的小例子吧。[出处](http://cynthia.blog.51cto.com/839408/850565)
我截取一点点来讲哈，因为之前我已经知道怎么执行了lua脚本了。  o(*￣▽￣*)o   
好了，为了学习方便，我们从lua5.1开始.( **(＃°Д°) 记住是lua5.1!!!不是lua5.2!!!!!老子刚学时候在这绕很久的弯路，尼玛版本不对，一堆范例运行都失效了!!!我次卧~老子在这里被打击得很受伤~抹泪~浪费了很多时间。**)
##1，编辑lua文件:  
>
    tao@tao:~/lua-5.1.0$ vi test.lua
在test.lua中输入以下代码后保存:
>
    print("-----------------------------");    
    print("Hello world"); 
    print("-----------------------------");
在命令行模式下输入以下命令
>
    tao@tao:~/lua-5.1.0$ lua test.lua 
    ----------------------------- 
    Hello world 
    --------------

##2，在C文件中嵌入lua代码  
>
    tao@tao:~/lua-5.1.0$ vi a.c
2.1在a.c文件中输入以下代码：
>
    #include <stdio.h> 
    #include <lua.h> 
    #include <lauxlib.h> 
    #include <lualib.h> 
     
    int main(int argc, char *argv[]) 
    { 
                    char line[BUFSIZ]; 
                    lua_State *L = luaL_newstate(); 
                    luaL_openlibs(L); 
                    //while (fgets(line, sizeof(line), stdin) != 0) printf("%s\n",line); 
      luaL_dofile(L, "test.lua"); 
                    lua_close(L); 
                    return 0; 
    }


在命令行模式下输入以下命令：

>
    tao@tao:~/lua-5.1.0$ gcc -o a a.c -I/usr/local/include/ -L/usr/local/lib/ -llua -lm -ldl 
    tao@tao:~/lua-5.1.0$ ./a 
    ----------------------------- 
    Hello world 
    -----------------------------

这时候高潮来了。如果你用的是5.2，就会出现一堆报错，说什么人家不认识luaopen啦
>
    a.c:(.text+0x23): undefined reference to `lua_open' 

这时候你只要修改lua_open函数为luaL_newstate函数再编译就可以了。
编译的命令你可以这样(一般编译)：
>
    gcc -o xxx xxx.c -llua -ldl -lm
还可以这样(指定lib地址，呵呵，我猜的，(￣▽￣") ):
>
    gcc -I/usr/local/include/ -L/usr/local/lib/ -llua a.c 
甚至这样(加上-lm是数学库，大写的那个不知道是干嘛，好像在某个操作手册看到过...(ˇˍˇ） ...忘了)：
>
    gcc -I/usr/local/include/ -L/usr/local/lib/ -lm -DLUA_USE_READLINE a.c /usr/local/lib/liblua.a

这个例子是C通过C API调用了lua，如果你觉得 矮油，改个函数就可以不用由lua5.2换到lua5.1了，那你就图样图森普了~ [摸摸头](～￣▽￣)ノ 

##下面这个是lua通过C API 调用C编写的动态链接库~ 
简单的话，你可以参考这个小[例子](http://blog.163.com/dang_wenyun/blog/static/42206525200911317247238/)  
其实我觉得上面这里例子已经很好很好了。但是呢，我还是要讲下我的小例子。<(￣ˇ￣)/ 因为这样才能显得我真的是实践过的。   
因为我想用lua写一个一致性hash,但是中间的murmurhash用lua写不了，我想过用加法hash来代替，但是加法hash得不均匀感觉。所以
就可以用C来写了，然后给lua调用就行了。  
vim hash64.c
>
    #include <math.h>
    #include "lua.h"
    #include "lualib.h"
    #include "lauxlib.h"
    size_t MurmurHash64B ( const void * key, int len, unsigned int seed )
    {
     const unsigned int m = 0x5bd1e995;
            const int r = 24;
            unsigned int h1 = seed ^ len;
            unsigned int h2 = 0;
    
            const unsigned int * data = (const unsigned int *)key;
    
            while(len >= 8)
            {
                    unsigned int k1 = *data++;
                    k1 *= m; k1 ^= k1 >> r; k1 *= m;
                    h1 *= m; h1 ^= k1;
                    len -= 4;
    
                    unsigned int k2 = *data++;
                    k2 *= m; k2 ^= k2 >> r; k2 *= m;
                    h2 *= m; h2 ^= k2;
                    len -= 4;
            }
    
            if(len >= 4)
            {
                    unsigned int k1 = *data++;
                    k1 *= m; k1 ^= k1 >> r; k1 *= m;
                    h1 *= m; h1 ^= k1;
                    len -= 4;
            }
    
            switch(len)
            {
            case 3: h2 ^= ((unsigned char*)data)[2] << 16;
            case 2: h2 ^= ((unsigned char*)data)[1] << 8;
            case 1: h2 ^= ((unsigned char*)data)[0];
                            h2 *= m;
            };
    
            h1 ^= h2 >> 18; h1 *= m;
            h2 ^= h1 >> 22; h2 *= m;
            h1 ^= h2 >> 17; h1 *= m;
            h2 ^= h1 >> 19; h2 *= m;
    
            size_t h = h1;
    
            h = (h << 32) | h2;
            return h;
    }
        static int l_sin (lua_State *L)
    {
     char *string = luaL_checkstring(L, 1);
     unsigned int d=MurmurHash64B(string,strlen(string)-1,0x1234ABCD);
    
    lua_pushnumber(L, d);
     return 1; /* number of results */
    }
    
    static const struct luaL_reg hashlib [] = {
     {"murmurhash64b", l_sin},
     {NULL, NULL} /* 必须以NULL结尾 */
    };
    
    int luaopen_hashlib (lua_State *L)
    {
     luaL_openlib(L, "hashlib", hashlib, 0);
     return 1;
    }

(我嚓，感觉代码略长啊，算了，你们还是去看我刚才说的那个小例子吧。那个比较简单点,真心的~)  
代码应该不难看懂吧？就是函数的互相调用而已。参考下[lua的文档](http://www.lua.org/pil)吧，反正就是注册命令啊，然后hashlib就是你调用时用的
对象神马的。那个命令再调用那个函数这样子。（这个看起来是不是有点像在写nginx的模块涅~(￣▽￣)~* ）  
哦，对了，有的murmurhash是有用uint64_t类型的（这里本来也有，当被我改了，因为当时年少无知，╮(￣▽￣")╭ ），你编译时候说无法识别的话，
只要加上头文件#include <stdint.h>就可以了。stlent警告的话当然就是加上string.h啦。这个应该地球人都懂的。然后啊，还有啊，murmurhash64A还会有这句case 7: h ^= (uint64_t)(data2[6]) << 48;
编译会说有错误，这时候把(uint64_t)(data2[6])改成((uint64_t)data2[6])就可以了。  
(～ o ～)~zZ 一下子偏题那么多，好累，感觉不会再爱了。  
然后编译一下:
>
    gcc  hash64.c  -fPIC --share -o libhash64.so
-fPIC代表和位置无关， --share 是说明共享，  .so表示这是一个动态链接库, 据说一般命名规则都是lib开头~  

编译完成你就会看到libhash64.so文件了。  
这时候我们在写一个lua文件去调用它就行了。记得把动态链接库包含进去就行了 。
>
    vi wbc.lua
>
    package.loadlib("/home/test/www/libhashlib.so", "luaopen_hashlib")()
    print(hashlib.murmurhash64b("foooooo"))

然后lua wbc.lua就可以了。当然，上面loadlib的时候可以不用绝对路径，但是这样就得把.so文件拷贝到/usr/lib  或/lib下面(系统寻找的路径)
了。感觉略麻烦。当然，等你的.so多了后，你就可以单独建一个文件夹，然后把path添加到$PATH里面，这样就可以（应该是的吧，反正我没试过...啦啦啦~）    
好了，你可以试试，输出结果应该是这样的：
>
    [root@qrwefsdf www]# lua hash.lua 
    1876582853

好了，今天就到这了。呼~好累啊~感觉不会再爱了。  

最后再link几个我觉得还可以的链接吧。  
[注释很详细的小实例](http://www.cnblogs.com/stephen-liu74/archive/2012/07/23/2469902.html)  
[lua函数的查询](http://www.lua.org/manual/5.1/manual.html#pdf-string.format)  
[这个实例主要看后面对C API的分析,可以当成源码分析的入门吧](http://blog.csdn.net/ym012/article/details/7188707)  
