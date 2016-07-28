title: jvm内存溢出的原因及解决方案介绍
categories: java
tags: java
---
内存溢出（OutOfMemory,oom）通常出现在某一内存空间块耗尽的时候。造成内存溢出的情况有：堆溢出，栈溢出，直接内存溢出,永久区溢出等。
<!--more-->
# 堆溢出
## 现象
当大量对象占用了堆空间，且都是强引用不能被回收时，如果对象大小之和大于了Xmx时，就会出现堆溢出。
堆溢出时，会提示 java heap space

## 解决方案
1.加大Xmx的值
2.利用visualvm工具，分析找到占用大量空间的对象，并进行优化

# 直接内存溢出
## 现象

例子：

    public class DirectBufferOOM {
        public static void main(String args[]) {
            for (int i = 0; i < 1024; i++) {
                ByteBuffer.allocateDirect(1024*1024);
                System.out.println(i);
                //System.gc();
            }
        }
    }

jvm配置如下：
```
-Xmx1g -XX:PrintGCDetails
```
上面的配置运行在jdk1.7 64位上时，不会出现问题。当运行在jdk1.7 32位上时，会出现OOM。原因是：32位系统进程的寻址空间为4G，其中2G为用户空间，2G为系统空间，所以实际可用的系统空间有2G。当java内存（堆空间、栈空间、直接内存空间、虚拟机自身所用的内存）之和大于2G时，就会出现OOM。

## 解决方案
可以设置 -XX:MaxDirectMemorySize的大小，比如512M。当直接内存达到这个值时，会出发垃圾回收。

# 栈溢出
## 现象
创建了太多的线程。unable to create new native thread. 说明创建的线程已经饱和。

## 解决方案
1.减小堆空间
2.减小每个线程所占用的栈空间。即减小-Xss的值。但是会增加栈溢出的风险。

# 永久区溢出
## 现象
当永久区存储的元数据类型太多时，会出现 Permgen space. 

## 解决方案
1.增加MaxPermSize
2.减少系统需要的类的数量
2.使用ClassLoader合理地装载各个类，并定期进行回收
