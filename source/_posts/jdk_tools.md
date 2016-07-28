title: jdk内置工具的使用
categories: java
tags: java
---

jdk内置工具对于查看监控java服务的运行情况、服务的性能问题分析十分有用。对于某些监控工具的运行，需要在服务器端配置参数，来方便客户端连接。
<!--more-->
# 设置远程监控的相关选项
需要在server.sh启动脚本中加入以下参数：
- 开启JVM远程监控 -Dcom.sun.management.jmxremote=true
- 监控的IP地址 -Djava.rmi.server.hostname=192.168.91.166，远程进程所在主机的IP。
- 监控的端口 -Dcom.sun.management.jmxremote.port=50013，这个端口值可以任意设置，但在之后用Jconsole连接这个远程进程的时候，远程进程中的port一定要和此处的设置一致，并且一定要和远程进程的服务端口区分开。
- 是否禁用ssl验证 -Dcom.sun.management.jmxremote.ssl，false为禁用，true为启用。
- 是否需要用户密码验证 -Dcom.sun.management.jmxremote.authenticate，false为不需要验证，true为需要验证。

# jdk 内置工具介绍
## jinfo
jinfo pid 查看进程的java系统参数以及vm参数.

## jmap
主要是查看堆的情况，也可以将堆信息dump下来。
### jmap -heap pid 
查看java堆的使用情况

```
Attaching to process ID 6196, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.91-b14

using thread-local object allocation.
Parallel GC with 2 thread(s)

Heap Configuration:
MinHeapFreeRatio         = 0                    ＃-xx:MinHeapFreeRatio 设置jvm堆最小空闲比率
MaxHeapFreeRatio         = 100                  #-xx:MaxHeapFreeRatio 设置jvm堆最大空闲比率
MaxHeapSize              = 536870912 (512.0MB)  ＃设置jvm堆的最大大小
NewSize                  = 89128960 (85.0MB)    ＃设置jvm堆的 新生代的默认大小
MaxNewSize               = 178782208 (170.5MB)  ＃设置jvm堆的 新生代的最大大小
OldSize                  = 179306496 (171.0MB)  ＃设置jvm堆的 老年代的大小
NewRatio                 = 2                    ＃设置新生代和老年代的大小比例
SurvivorRatio            = 8                    ＃设置新生代中的 eden区和survivor区的大小比值
MetaspaceSize            = 21807104 (20.796875MB)＃jdk1.8，类似于jdk1.7中的 permsize
CompressedClassSpaceSize = 1073741824 (1024.0MB) 
MaxMetaspaceSize         = 17592186044415 MB
G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
capacity = 82837504 (79.0MB)
used     = 27614056 (26.334815979003906MB)
free     = 55223448 (52.665184020996094MB)
33.33521010000494% used
From Space:
capacity = 3145728 (3.0MB)
used     = 1534048 (1.462982177734375MB)
free     = 1611680 (1.537017822265625MB)
48.766072591145836% used
                                                            
To Space:
capacity = 3145728 (3.0MB)
used     = 0 (0.0MB)
free     = 3145728 (3.0MB)
0.0% used
PS Old Generation
capacity = 201850880 (192.5MB)
used     = 49296248 (47.01256561279297MB)
free     = 152554632 (145.48743438720703MB)
24.422112006645698% used

26112 interned Strings occupying 2975608 bytes.
```

### jmap -histo pid 
查看堆内存中对象数量和大小
对比：jmap -histo:live pid jvm会先出发gc，再进行统计

```
num     #instances      #bytes  class name
----------------------------------------------
1:         52843       25434824  [B
2:        204331       23624576  [C
3:         26098        7198896  [I
4:        116142        4724056  [Ljava.lang.Object;
5:        161108        3866592  java.lang.String
6:         27160        2390080  java.lang.reflect.Method
7:         47005        1504160  java.io.ObjectStreamClass$WeakClassKey
8:         45639        1460448  java.util.HashMap$Node
9:         12598        1443432  [Ljava.util.HashMap$Node;
10:        24589        1180272  java.util.HashMap
```

### jmap -dump:format=b,file=heapdump pid
将当前内存使用的详细信息存储到文件中

## jstack
打印线程的栈信息
参考：http://www.blogjava.net/jzone/articles/303979.html

## jstatd
为了使jstat,jps命令能远程监控服务器上的java服务，需要在服务器上运行jstatd服务。方式如下：
首先新建一个文件，假设目录和文件名是 /home/jstatd.policy  文件内容如下：
    
    grant codebase "file:${java.home}/../lib/tools.jar" {  
        permission java.security.AllPermission;  
    };

然后启动jstatd服务。
    
    jstatd -p 8725 -J-Djava.security.policy=/home/jstatd.policy -J-Djava.rmi.server.hostname=XX.XXX.X.XX

## jstat
jstat 参数比较多，主要有以下几类：
- 类的加载、卸载情况
- 查看新生代、老年代和持久区的容量及使用情况
- 查看新生代、老年代和持久区的垃圾收集情况，包括垃圾回收的次数以及垃圾回收所占用的时间
- 查看新生代中eden区和survivor区中容量及分配情况

参考：http://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html

jstat远程监控命令：
    
    jstat -gcutil 18182@10.94.96.146:1099 1000

## jps
显示当前运行的java进程以及相关参数。实现原理是：读取 /tmp/hsperfdata 有几个文件，文件名是以java服务进程的ID命名的。

jps远程监控命令：
    
    jps -lv rmi://10.94.96.146:1099 1099为jstatd的端口

## jvisualvm
是一种可视化的监控工具。包括 cpu、堆、线程、类的监控，也可以分析dump的堆文件，支持类sql查询。

## jconsole
是一种可视化的监控工具。包括cpu、内存、线程、类的监控。
