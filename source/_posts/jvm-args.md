title: jvm常用参数设置
categories: java
tags: java

---
jvm的参数包括：跟踪调试参数、堆参数、栈参数、方法区配置参数、直接内存参数和虚拟机的工作模式。
<!--more-->
# 跟踪调试参数
## 跟踪垃圾收集
-XX:+PrintGC,在虚拟机启动后，遇到GC，就会打印日志
-XX:+PrintGCDetail 打印详细的GC日志.是虚拟机在退出前打印堆得详细信息，详细信息描述了当前堆得各个区的使用情况。
-XX:+PrintHeapAtGC 在GC日志输出前后，都有详细的堆信息输出
-XX:+PrintGCTimeStamps 输出GC发生的时间，该时间为虚拟机启动后的时间偏移量
-XX:+PrintGCApplicationConcurrentTime 打印应用程序的执行时间
-XX:+PrintGCApplicationStoppedTime 打印应用程序由于GC而产生的停顿时间
-XX:+PrintReferenceGC 跟踪系统的软引用、弱引用、虚引用和Finallize队列

## 跟踪类的加载、卸载
-verbose:class 跟踪类的加载和卸载
-XX:+TraceClassLoading 跟踪类的加载
-XX:+TraceClassUnloading跟踪类的卸载

## 系统参数查看
-XX:+PrintVMOptions 打印虚拟机接受到的命令行显示参数
-XX:+PrintCommandLineFlags 打印传递给虚拟机的显示和隐示参数，隐示参数是虚拟机启动时自行设置的。

# 堆参数配置

![堆参数配置示意图](https://raw.githubusercontent.com/buptlsy/images/gh-pages/heap-malloc-struct.png)

# 非堆内存的参数配置
## 方法区参数配置
在jdk1.7和jdk1.6中，方法区的配置参数是：-XX:PermSize 指定初始的永久区大小，-XX:MaxPermSize 指定最大永久区
在jdk1.8，永久区被移除，使用了元数据区存放类的元数据，元数据区只受系统可用内存大小的限制，但是可以通过-XX:MaxMetaspaceSize来指定永久区的最大可用值。

## 栈参数配置
-Xss 指定线程的栈大小

## 直接内存参数配置
直接内存跳过了java堆，可以直接访问原生堆空间。最大可用直接内存可以使用 -XX:MaxDirectMemeorySize 来设置，如不设置，默认值为最大堆空间，即-Xmx。当直接内存使用量达到最大值时，会出发垃圾收集。

## jvm工作模式
-client 指定client工作模式
-server 指定server工作模式
-version 查看工作模式是哪一种

server模式启动比较慢，因为会收集更多的性能参数，进行更多的性能优化
