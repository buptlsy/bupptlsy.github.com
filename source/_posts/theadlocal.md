title: threadlocal的那些事儿
categories: java
tags: java
---
threadlocal是另外一种解决多线程并发问题的方案。其主要思想是在每个线程中维护一份共享变量的副本。
<!--more-->
# 问题的由来

看下面日期转换的代码：

    public class ThreadLocalTest1 {
        private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

        static class PrintDate implements Runnable {
            private int i;

            public PrintDate(int i) {
                this.i = i;
            }

            @Override
            public void run() {
                try {
                    Date date = simpleDateFormat.parse("2016-07-21" + " " + "10:10:" + i % 60);
                    System.out.println(date);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
                                                                                                                                           public static void main(String[] args) {
            Executor executor = Executors.newCachedThreadPool();
            for (int i = 0; i < 10; i++) {
                executor.execute(new PrintDate(i));
            }
        }
    }

上面的代码运行会出现抛出异常：
```
java.lang.NumberFormatException: multiple points
```
这是因为多线程访问parse方法时，会对SimpleDateFormat的内部变量造成不确定的修改，从而会抛出异常。
解决方法可以是加锁 或者是 threadlocal。加锁的话，性能会造成影响。threadlocal是以空间换时间的策略。改变后的代码如下：

    public class ThreadLocalTest {

        private static ThreadLocal<SimpleDateFormat> simpleDateFormats = new ThreadLocal<SimpleDateFormat>() {
            @Override
            protected SimpleDateFormat initialValue() {
                return new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
            }
        };

        static class PrintDate implements Runnable {
            private int i;

            public PrintDate(int i) {
                this.i = i;
            }

            @Override
            public void run() {
                try {
                    SimpleDateFormat simpleDateFormat = simpleDateFormats.get();
                    Date date = simpleDateFormat.parse("2016-07-21" + " " + "10:10:" + i % 60);
                    System.out.println(date);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                                                                                                                                               }
                                                                                                                                                                                                                                                
        }
        
        public static void main(String[] args){                   
            Executor executor = Executors.newCachedThreadPool();
            for (int i = 0; i < 10; i++) {
                executor.execute(new PrintDate(i));
                                                                                                                                                }
                                                                                                                                           }

    }

# threadlocal的原理
threadlocal的原理，就是每个线程维护一份共享变量的副本。其内部实现图，如下：

![threadlocal结构图](https://raw.githubusercontent.com/buptlsy/images/gh-pages/threadlocal-struct.png)

## threadlocalmap的内部实现
ThreadLocalMap是自定义实现的hashmap，仅仅为了维护threadlocal的值。
threadlocalMap的实现和hashmap的实现有很大的差别。threadlocalmap只是单纯的用数组实现，当出现冲突时，利用 线性探测法 来找到空的位置插入。而且它通过弱引用和在get、set方法中处理key为null的途径，极好的防止了内存泄漏的问题。

[ThreadLocal代码的部分注释](https://github.com/buptlsy/java/blob/master/jdkSourceCode/ThreadLocal.java)

