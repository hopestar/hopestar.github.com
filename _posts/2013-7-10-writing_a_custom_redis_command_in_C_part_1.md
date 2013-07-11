---
layout: default
title: 用C编写自定义的redis命令- part-1
---

不久前，我们发现自己在需要一个ZDIFF 的Redis命令。我们可以像我们其他的代码一样在Lua中实现，但为什么不直接在Redis中实现呢？ Redis的伟大的事情之一是有一个干净的代码库，即使对于像我这样的一个非常生疏的C程序员来说。

你会想要做的第一件事是定义你的命令。在redis.h底部添加函数名称:

      void xdiffCommand(redisClient *c);

在redis.c，我们将注册这个function。近顶部，你会发现完整的的redisCommandTable。首先，我们将注册我们的命令，一会儿我会解释每个参数的作用。所以，在这个变量的底部，添加一个新条目：

      {"xdiff",xdiffCommand,5,"r",0,NULL,1,2,0,0,0},

这个条目的具体含义是:
      
      * name: a string representing the command name.
      * 命令的名称
      *
      * function: pointer to the C function implementing the command.
      * 命令的实现函数
      *
      * arity: number of arguments, it is possible to use -N to say >= N
      * 参数的数量，可以用 -N 表示 >= N
      *
      * sflags: command flags as string. See below for a table of flags.
      * 字符串形式的 FLAG ，用来计算下面的 flags 属性
      *
      * flags: flags as bitmask. Computed by Redis using the 'sflags' field.
      * 位掩码形式的 FLAG ，由 sflags 字符串计算得出
      *
      * get_keys_proc: an optional function to get key arguments from a command.
      * This is only used when the following three fields are not
      * enough to specify what arguments are keys.
      * 一个可选的函数，用于从命令中取出 key 参数。
      * 只在以下三个参数不足以表示 key 参数时使用。
      *
      * first_key_index: first argument that is a key
      * 第一个是 key 的参数
      * last_key_index: last argument that is a key
      * 最后一个是 key 的参数
      * key_step: step to get all the keys from first to last argument. For instance
      * in MSET the step is two since arguments are key,val,key,val,...
      * 从 first 参数和 last 参数间，获取所有 key 的步数（step）
      * 比如说， MSET 命令的格式为 MSET key value [key value ...]
      * 它的 step 就为 2
      *
      * microseconds: microseconds of total execution time for this command.
      * 执行这个命令耗费的总微秒数
      *
      * calls: total number of calls of this command.
      * 命令被执行的总次数

最后两个0是用于redis内部跟着统计命令信息的。
1,2,1分别表示第一个key是参数1,最后一个key是参数2（注意这些都是key值哟，而不是其它参数哟），取值步长为1. 举个栗子，我们有个命令要交换3个键值对：
key1 value1 key2 value2 key3 value3, 我们要用： 1,5,1 来表示。5是表示最后一个key值是第5个参数。

事实上你还不清楚xdiff能干吗，我们会拿一个有序的集合，从给定的偏移量开始，找到的不在该集合的第一个计数值。例如，我们可以用它做这样的事情：

      zadd duncan:friends 101 murbella 100 leto 95 paul 85 teg 85 gurney 80 chaini 70 thufir 60 leto2
      
      sadd family:atreides ghanima paul jessica leto leto2
      
      xdiff duncan:friends family:atreides 1 2
      
      #skips murbella since the offset is 1, and not 0
      output: teg gurney

我们将添加一个新的的文件，在src目录创建文件custom.c（尽可能的保持我们代码的独立，方便代码每次改变后从主项目pull下来比较容易）。接下来，在src/ Makefile中找到REDIS_SERVER_OBJ,在行尾加custom.o . 最后，打开我们新创建的的文件：custom.c，并添加以下内容：

      #include "redis.h"
      
      void xdiffCommand(redisClient *c) {
      
      }

您现在应该能够编译Redis了（通过make）。

结构体redisClient表示正在执行的上下文(context) ,web程序员可能会认为它是请求和响应。 我们不会直接处理它，尽管我们会将它传递给现有的功能。例如，我们想要做的第一件事是添加一些验证，以确保我们的所有输入都ok（由于我们对Redis预计5个参数，它将验证我们输入的基本变量）。

开始简单，我们可以使用getLongFromObjectOrReply的的函数来获得offset和count：

      long offset, count;
      
      if ((getLongFromObjectOrReply(c, c->argv[3], &offset, NULL) != REDIS_OK)) { return; }
      if ((getLongFromObjectOrReply(c, c->argv[4], &count, NULL) != REDIS_OK)) { return; }

这两行告诉Redis的第三和第四参数以long类型加载到我们的变量offset和count。如果失败，Redis的会回复一个错误信息（或者我们可以在第四个参数指定自定义错误消息）。我们可以默认我们的变量（比如0和10），但让我们先用简单的退出吧。

我们也希望载入我们的有序集和集合（argv[1]和argv[2]），并希望确保它们是正确的类型：
    
      robj *zobj, *sobj;
      
      zobj = lookupKeyReadOrReply(c, c->argv[1], shared.czero);
      if (zobj == NULL || checkType(c, zobj, REDIS_ZSET)) { return; }
      
      sobj = lookupKeyReadOrReply(c, c->argv[2], shared.czero);
      if (sobj == NULL || checkType(c, sobj, REDIS_SET)) { return; }

和之前差不多。我们定义了两个redis 对象，zobj SOBJ，并尝试从给定的参数中加载它们。如果他们是null或错误的类型则退出。

redisObjects（robj）是基本的对象结构。它包装底层的数据结构（可通过的void * ptr的成员），并提供了一个类型，checkType依赖于一个引用计数（int refcount）和LRU值（为过期准备的，我猜滴），还有其他成员（其中一部分我们将在下一个部分详细探讨）。
下面是redisObjects（robj）的结构:

    typedef struct redisObject {
        unsigned type:4;
        unsigned storage:2;     /* REDIS_VM_MEMORY or REDIS_VM_SWAPPING */
        unsigned encoding:4;
        unsigned lru:22;        /* lru time (relative to server.lruclock) */
        int refcount;
        void *ptr;
        /* VM fields are only allocated if VM is active, otherwise the
         * object allocation function will just allocate
         * sizeof(redisObjct) minus sizeof(redisObjectVM), so using
         * Redis without VM active will not have any overhead. */
    } robj;

今天的最后一点，仅仅是当我们验证通过时能返回一个虚拟的答案：

      #include "redis.h"
      
      void xdiffCommand(redisClient *c) {
        long offset, count;
        robj *zobj, *sobj;
      
        if ((getLongFromObjectOrReply(c, c->argv[3], &offset, NULL) != REDIS_OK)) { return; }
        if ((getLongFromObjectOrReply(c, c->argv[4], &count, NULL) != REDIS_OK)) { return; }
      
        zobj = lookupKeyReadOrReply(c, c->argv[1], shared.czero);
        if (zobj == NULL || checkType(c, zobj, REDIS_ZSET)) { return; }
      
        sobj = lookupKeyReadOrReply(c, c->argv[2], shared.czero);
        if (sobj == NULL || checkType(c, sobj, REDIS_SET)) { return; }
      
        addReplyLongLong(c, 9001);
      }

你可以返回并make下代码。接下来，运行redis（./src/redis-server），通过客户端redis-cli 连接到它。您可以尝试传递各种xfind的参数，以确保它的所有工作正常（不用担心，在某些时候，我们会适当增加测试）。

在接下来的部分，我们将构建xdiff的核心代码。

原文:[http://openmymind.net/Writing-A-Custom-Redis-Command-In-C-Part-1/]

