---
layout: default
title: Writing a Custom Redis Command In C - Part 2
---
在第1部分我们建立了xfind的方法和并将其嵌入redis。现在是时候写实际执行部分的代码了。提醒一下，我们的目标是得到一个有序集合，从中减去一组，并提供分页（偏移/计数）。

lookupKeyReadOrReply函数是用来返回一个redis的对象（robj），我们简要介绍。 robj有一个我们感兴趣的成员：encoding。你看，在Redis的大多数数据结构有多个可能的实现。例如，一个有序的集合，可以是一个skiplist或ziplist的。 根据Redis的所存储的数据（值的类型或数量），挑选其中一种实现。它也可以实现时很好的根据需要进行转换。

不幸的是，这让情况变得更复杂。如果您通过浏览Redis的代码库，你会发现许多行类似的代码:if (r->encoding == XYZ) { ... } else { ... }.在许多情况下，它们将封装作为实现细节的抽象——尽管他们并不总是公开提供。尽管我们很容易修改访问性，但我们会采取不同的方法，而不是对作为通用的数据结构的有序集合和集合再次编码，我们会针对特定的实现进行编码。现在已经有一些Redis的命令是这样做了。例如，如果你对有序集进行排序，Redis的将它转换到skiplist，而这正是我们要做的：
      zobj* zobj = lookupKeyReadOrReply(c, c->argv[1], shared.czero);
      if (zobj == NULL || checkType(c, zobj, REDIS_ZSET)) { return; }
    
      zsetConvert(zobj, REDIS_ENCODING_SKIPLIST);
      zset *zset = zobj->ptr;

同样，在只存储int类型的情况下，集合（我们用来比较滴那个）会使用一种特殊的编码​​。不过由于我们的集合将包含字符串，所以我们可以放心地假定，我们集编码是一个哈希表，而不是一个整数集合中。

     sobj = lookupKeyReadOrReply(c, c->argv[2], shared.czero);
      if (sobj == NULL || checkType(c, sobj, REDIS_SET)) { return; }
    
      dict *diff = (dict*)sobj->ptr;

我们对skiplist的实现再次的好处之一是我们在项目里可以轻易回撤（原文:A nice thing about the skiplist implementation we are programming against is the ability to easily walk forwards or backwards through the items.）。现在，我们只关心我们如何向后移动，但这个需要稍微添加一个排序参数，我们首先得到skiplist的尾部，然后向后移动：
     zskiplist *zsl = zset->zsl;
      zskiplistNode *ln = zsl->tail;
    
      while(ln != NULL) {
        //todo
        ln = ln->backward;
      }

完整的循环，包括申请diff,它看起来像:

    long found = 0, added = 0;
      while(ln != NULL) {
        robj *item = ln->obj;
        if (dictFind(diff, item) == NULL && found++ >= offset) {
          addReplyBulk(c, item);
          if (++added == count) { break; }
        }
        ln = ln->backward;
      }

它可以分解为以下几步:1，从当前的元素里取出值。判断这个item是不是在我们的字典里，如果不在且超出了offset，我们就在replay中添加item。所有都添加完了，就跳出循环。

返回需要值的数量作为前缀，我们预先不知道该值多少（可以比count小）。Redis的提供了一个函数来分配空间的长度，我们在后面写上。
    void *replylen = addDeferredMultiBulkLength(c);
      while(ln != NULL) {
        //...
      }
      setDeferredMultiBulkLength(c, replylen, added);
    


各位亲，到这里，就是一个量身定制的xdiff的简单实现了，这是所有类型的改进方法。我们可以添加一个命令的参数或实现类似排序的GET的功能。
我们做的最后一个改进是直接跳到offset，。因为我们知道，我们不会找到一个适合我们的页面直到item。我们可以简单地使用的zslGetElementByRank功能，而不是指向集合的尾部。我们完整的代码看起来像：

    #include "redis.h"void xdiffCommand(redisClient *c) {
      long offset, count, found = 0, added = 0;
      robj *zobj, *sobj;
      zset *zset;
      dict *diff;
      void *replylen;
    
      if ((getLongFromObjectOrReply(c, c->argv[3], &offset, NULL) != REDIS_OK)) { return; }
      if ((getLongFromObjectOrReply(c, c->argv[4], &count, NULL) != REDIS_OK)) { return; }
    
      zobj = lookupKeyReadOrReply(c, c->argv[1], shared.czero);
      if (zobj == NULL || checkType(c, zobj, REDIS_ZSET)) { return; }
    
      sobj = lookupKeyReadOrReply(c, c->argv[2], shared.czero);
      if (sobj == NULL || checkType(c, sobj, REDIS_SET)) { return; }
    
      zsetConvert(zobj, REDIS_ENCODING_SKIPLIST);
      zset = zobj->ptr;
      diff = (dict*)sobj->ptr;
    
      long zsetlen = dictSize(zset->dict);
      zskiplistNode *ln = zslGetElementByRank(zset->zsl, zsetlen - offset);
    
      replylen = addDeferredMultiBulkLength(c);
      while(ln != NULL) {
        robj *item = ln->obj;
        if (dictFind(diff, item) == NULL) {
          addReplyBulk(c, item);
          if (++added == count) { break; }
        }
        ln = ln->backward;
      }
      setDeferredMultiBulkLength(c, replylen, added);
    }

也许这个实现过于具体，但希望你能从中领悟到如何建立一个你自己的命令。

[原文链接](http://openmymind.net/Writing-A-Custom-Redis-Command-In-C-Part-2/)

