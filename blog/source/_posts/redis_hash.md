---
title: 字典的实现
categories: redis
tags: redis

---
redis字典的实现在某些地方的设计很类似于jdk中hashmap的设计，如解决冲突方面都是利用了链式法解决冲突的。在存储结构和rehash方案上，redis的hash实现明显比hashmap更有优势。
redis字典实现：
<!--more-->

### 存储结构

字典底层的存储结构图如下：
![hash存储结构图](https://raw.githubusercontent.com/buptlsy/images/master/redis-hash.png)

    typedef struct dict {
        dictType* type; // 特定函数的类型
        void* privdata; //特定函数的参数
        dictht* ht; //哈希表
        int rehashidx; //rehash进度。-1表示未开始
    } dict;

    typedef struct dictType {
    } dictType;

    typedef struct dictht {
        unsigned long size; //哈希表大小
        unsigned long sizemask; //掩码，用来计算位置
        unsiged long used; //已经使用的数量
        dictEntry **table; //哈希表数组
    } dictht;

    typedef struct dictEntry {
        void* key; // key
        union {
            void *val;
            uint64_t u64;
            int64_t s64;
        } v; // value
        struct dictEntry *next; //形成链表
    } dictEntry;

### 哈希函数

字典计算出每个key最终放的位置是通过：
    
    // 通过哈希函数计算出哈希值
    hash = dict->type->hashFunction(key);
    // 通过哈希值计算出最终索引的位置
    index = hash & dict->ht[x].sizemask;

哈希函数用的是：murmurhash2

### 解决冲突

当多个key计算出的索引值一样的话，通过拉链法解决冲突。

### rehash

#### 进行rehash的条件

扩容：
1. 当服务器没有执行bgsave和bgrewriteaof时，负载因子大于等于1
2. 当服务器执行bgsave或者bgrewriteaof时，负载因子大于等于5
(负载因子=哈希表已用大小/哈希表大小)

为什么会有这样的区分？


缩容：当负载因子小于0.1时

#### rehash过程

1. 确定ht[1]哈希表的大小
如果执行的是扩展操作， 那么 ht[1] 的大小为第一个大于等于 ht[0].used * 2 的 2^n （2 的 n 次方幂）；
如果是缩容，那么 ht[1] 的大小为第一个大于等于 ht[0].used 的 2^n 。
2. 将ht[0]的kv迁移到ht[1]
利用渐进式迁移策略。
步骤：
a. 为 ht[1] 分配空间， 让字典同时持有 ht[0] 和 ht[1] 两个哈希表。
b. 在字典中维持一个索引计数器变量 rehashidx ， 并将它的值设置为 0 ， 表示 rehash 工作正式开始。
c. 在 rehash 进行期间， 每次对字典执行添加、删除、查找或者更新操作时， 程序除了执行指定的操作以外， 还会顺带将 ht[0] 哈希表在 rehashidx 索引上的所有键值对 rehash 到 ht[1] ， 当 rehash 工作完成之后， 程序将 rehashidx 属性的值增一。
d. 随着字典操作的不断执行， 最终在某个时间点上， ht[0] 的所有键值对都会被 rehash 至 ht[1] ， 这时程序将 rehashidx 属性的值设为 -1 ， 表示 rehash 操作已完成。
3. 释放ht[0], 将ht[1]设置为ht[0]，并在ht[1]上申请空的哈希表

