---
title: countDownLatch的使用和实现
categories: java
tags: java
---
countDownLatch实现了一种同步策略，允许一个或者多个线程等待直到一组操作被其他线程执行完成。
<!--more-->
### countDownLatch实现源码分析：

    public class CountDownLatch {
        //利用AQS实现Sync类，来表示count
        private static final class Sync extends AbstractQueuedSynchronizer {
            Sync(int count) {
                setState(count);
            }
            
            int getCount() {
                return getCount();
            }
            
            protected int tryAcquireShared(int acquires) {
                return (getState == 0) ? 1 : -1;
            }

            protected boolean tryReleaseShared(int releases) {
                for (;;) {
                    int c = getState();
                    if (c == 0) {
                        return false;
                    }
                    int nextc = c - 1;
                    if (compareAndSetState(c, nextc)) {
                        return nextc == 0;
                    }
                }
            }
        }

        private final Sync sync;

        /*
        * count：在线程可以通过之前，countDown()触发的次数。
        */
        private CoundDownLatch(int count) {
            if (count < 0) throws new IlleagalArgumentException("count < 0");
            this.sync = new Sync();
        }

        /*
        * 如果当前count减小到0，那么方法立即返回
        * 如果当前count比0更大，那么当前线程阻塞。直到count减小到0或者一些其他的线程中断了当前线程
        *  
        */
        public void await() throws InterruptedException {
            sync.acquireSharedInterruptibly(1);
        }
        
        /**
        * 引起当前线程等待直到门栓值减到了0。除非这个线程被中断或者超过了指定的等待时间 。如果当前count=0，那么这个方法立即返回true。如果count>0,线程变为等待，直到下面3件事情发生：
        * 1.count值变为0
        * 2.超过了给定的等待时间
        * 3.其他的线程中断了当前线程。如果当前线程有着中断状态设置在方法入口，或者当等待的时候被中断，那么会抛出InterruptedException并且当前线程的中断状态被清楚。
        /
        public boolean await(long time, TimeUnit unit) throws InterruptedException {
            return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
        }

        /**
         * 减少门栓的count值。如果count减到0，释放所有的等待线程。
         * 如果当前count值大于0，那么count减1；如果减少后的count值为0，则所有的等待线程变为可运行状态；如果当前的count为0，那么什么也不发生。
         */
        public void countDown() {
            sync.releaseShared(1);
        }

        /**
        * 返回当前count值。
        * 这个方法主要是为了测试或者调试。
        /
        public long getCount() {
            return sync.getCount();
        }
    }


### countDownLatch的使用例子：
典型用法 例子1：将一个问题分为N个部分，把每一部分描述为一个Runnable,并且减小门栓值，将所有的Runnables放进一个Executor中。当所有子部分完成时，协作线程将能够通过await。

    class Driver {
        void main() throws InterruptedException {
            CountDownLatch doneSignal = new CountDownLatch(N);
            Executor e = ...
            for (int i = 0; i < N; i++) {
                e.execute(new WorkerRunnable(doneSignal, i));
            }
            doneSignal.await(); //等待所有的线程结束
        }
    }

    class WorkerRunnable implements Runnable {
        private final CountDownLatch doneSignal;
        private int i;
        WorkerRunnable(CountDownLatch doneSignal, int i) {
            this.doneSignal = doneSignal;
            this.i = i;
        }

        public void run() {
            try {
                doWork(i);
                doneSignal.countDown();
            } catch (InterruptedException e) {
            
            }
        }

        void doWork() {...}
    }
    

典型用法 例子2：第一个开始信号阻止工作线程通过指导driver是准备好让他们通过；第二个结束信号是等到所有worker结束了，driver才能运行。

    class Driver {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(N);

        for (int i = 0; i < N; i++) {
            new Thread(new Worker(startSignal, doneSignal)).start();
        }
        doSomethingElse();
        startSignal.countDown();
        doSomethingElse();
        doneSignal.await();
    }
    
    class Worker implements Runnable {
        private final CountDownLatch startSignal;
        private final CountDownLatch doneSignal;
        Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
            this.startSignal = startSignal;
            this.doneSignal = doneSignal;
        }
        public void run() {
            try {
                startSignal.await();
                doWork();
                doneSignal.countDown();
            } catch (InterruptedException e) {
            
            }
        }
        void work() {...}
    }


问题：
1.TimeUnit类的使用和原理。
2.Memory consistency effects: Until the count reaches
 zero, actions in a thread prior to calling
 {@code countDown()}
 <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 actions following a successful return from a corresponding
 {@code await()} in another thread.
