---
title: abstractQueuedSynchronizer详解
categories: java
tags: java
---
abstractQueuedSynchronizer提供了一个依赖于先进先出的等待队列实现了阻塞锁和同步。这个类是利用一个表示状态的int值来为各种同步设计的。子类必须定义protect方法来改变状态（状态表示这个object是被获取还是释放）。其他的方法实现了所有的排队和阻塞机制。它支持排他锁和共享锁两种模式。例如读写锁。
<!--more-->
例子1：（利用abstractQueuedSynchronizer来实现互斥锁）
    class Mutex implements Lock, Serializable {
        private static class Sync extends AbstractQueuedSynchronizer {
            protected boolean isHeldExclusively() {
                return getState() == 1;
            }
            public boolean tryAcquire(int acquires) {
                assert acquires == 1;
                if (compareAndSetState(0, 1)) {
                    setExclusiveOwnerThread(Thread.currentThread());
                    return true;
                }
                return false;
            }
            protected boolean tryRelease(int release) {
                assert release == 1;
                if (getState() == 0) throw new IllegalMonitorStateException();
                setExclusiveOwnerThread(null);
                setState(0);
                return true;
            }
            Condition newCondition() {
                return new ConditionObject();
            }
            private void readObject(ObjectInputStream s) 
                throws IOException, ClassNotFoundException {
                s.defaultReadObject();
                setState(0);
            }
        }

        private final Sync sync = new Sync();
        public void lock() {
            sync.acquire(1);
        }
        public boolean tryLock() {
            return sync.tryAcquire(1);
        }
        public void unLock() {
            sync.release(1);
        }
        public boolean isLocked() {
            return sync.isHeldExclusively();
        }
        public Condition newCondition() {
            return sync.newCondition();
        }
        public boolean hasQueuedThreads() {
            return sync.hasQueuedThreads();
        }
        public void lockInterruptibly() throw InterruptedException {
            sync.acquireInterruptibly(1);
        }
        public boolean tryLock(long timeout, TimeUnit unit) 
            throws InterruptedException {
            return sync.tryAcquireNanos(1, unit.toNanos(timeout));
        }
    }


例子2：
    
