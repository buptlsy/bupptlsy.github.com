---
title: wait,notify,sleep
categories: java
tags: java

---
### wait与notify的配合

在java中，进程同步是利用synchronized()实现的。java的object类型中，带有一个内存锁。如果一个线程获取该内存锁后，其他线程便无法访问该内存。只有获取了Object的对象锁后，才能使用该对象的wait和notify。可以通过synchronized()和lock来获得内存锁。以synchronized为例，synchronized(obj)获得当前对象的锁；synchronized(class)来获得类的锁，保证在一个时刻只有一个线程可以访问这个类的实例。
<!--more-->
![线程状态图](https://raw.githubusercontent.com/buptlsy/images/master/threads-java.gif)
1. wait：线程获得对象锁后，主动释放对象锁，同时本线程休眠。
2. notify：唤醒等待的线程。notify调用后，并不是马上释放对象锁，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，jvm会在wait()对象锁的线程中随机选取一线程，获取对象锁，唤醒线程，继续执行。

### wait和sleep的区别

1. Thread.sleep(), Object.wait()
2. 两者都是暂停当前线程，释放cpu控制权
3. wait在释放cpu时，同时释放了对象锁的控制
