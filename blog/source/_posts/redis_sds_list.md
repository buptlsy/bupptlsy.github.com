---
title: redis的动态字符串和链表
categories: redis
tags: redis

---
redis的两个基本的数据结构：动态字符串和链表
<!--more-->
### 动态字符串（sds）

#### 结构

sds的结构主要包括：free(未使用空间的数量)，len(已使用空间的数量),buf(存储字符串的地方)。
![sds结构](https://raw.githubusercontent.com/buptlsy/images/master/redis-sds.png)
在数据的结尾会加上'\0'

#### 好处

1. 常量时间复杂度获取字符串大小。因为可以直接从len变量中取到值。
2. 避免缓冲区溢出。
类比c语言的strcat()函数，strcat在拼接字符串时，有可能会出现缓冲区溢出的情况。(strcat(dest, src), 如果dest字符串的剩余空间大小不足以放下src，那么src会覆盖原有的内容，导致缓冲区溢出)。
sds的拼接函数，会在做拼接操作前，看看dest是否有足够的free空间（至于怎样分配，请看第3条）。
3. 在动态增加和减少字符串时，减小分配内存的次数
出现的两种问题：
（1）缓冲区溢出。第2点介绍的内容。
（2）内存泄漏。如果不回收不用的空间，会出现内存泄漏的问题。
解决办法：
（1）扩展字符串时，分配内存空间的策略--空间预分配策略
当分配后的内存空间小于1M，会将free和len设置为相同的值；
当分配后的内存空间大于1M，则把free的值设为1M。
（2）剪断字符串时，会回收不用的空间--惰性回收策略
只是简单的将不用的空间数量加到free上。
这两种策略都是减少了内存分配次数。
4. 二进制安全
redis用buf数组来保存二进制数据。主要是因为它不是用'\0'来标识结尾，而是用len来判断字符串是否结束。

#### 主要操作

### 链表(list)

list的实现是通过双链表实现的。

### 结构

节点：
    
    typedef struct listNode {
        //前置节点
        struct listNode *prev;
        //后置节点
        struct listNode *next;
        //节点的值
        void *value;
    }

链表：
    
    typedef struct list {
        //表头节点
        listNode *head;
        //表尾节点
        listNode *tail;
        //节点个数
        unsigned long len;
        //节点复制函数
        void *(*dup)(void *ptr);
        //节点释放函数
        void *(*free)(void *ptr);
        //节点值对比函数
        void *(*match)(void *ptr, void *key);
    }


![list结构](https://raw.githubusercontent.com/buptlsy/images/master/redis-linkedlist.png)

#### 好处

1. 双链表的好处
2. len的好处
3. 多态。链表使用void* 保存节点值，并且可以通过dup、free、match三个属性为节点值设置类型特定的函数，所以链表可以用来保存各种不同类型的值。

#### 主要操作
