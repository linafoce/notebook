### CountDownLatch的使用

CountDownLatch是同步工具类之一，可以指定一个计数值，在并发环境下由线程进行减1操作，当计数值变为0之后，被await方法阻塞的线程将会唤醒，实现线程间的同步。

```java
public void startTestCountDownLatch() {
   int threadNum = 10;
   final CountDownLatch countDownLatch = new CountDownLatch(threadNum);
	//主线程启动10个子线程后阻塞在await方法，需要等子线程都执行完毕，主线程才能唤醒继续执行。
   for (int i = 0; i < threadNum; i++) {
       final int finalI = i + 1;
       new Thread(() -> {
           System.out.println("thread " + finalI + " start");
           Random random = new Random();
           try {
               Thread.sleep(random.nextInt(10000) + 1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println("thread " + finalI + " finish");

           countDownLatch.countDown();
       }).start();
   }

   try {
       countDownLatch.await();
   } catch (InterruptedException e) {
       e.printStackTrace();
   }
   System.out.println(threadNum + " thread finish");
}
```

### 构造器

CountDownLatch和ReentrantLock一样，内部使用Sync继承AQS。构造函数很简单地传递计数值给Sync，并且设置了state

```java
Sync(int count) {
    setState(count);
}
```

上文已经介绍过AQS的state，这是一个由子类决定含义的“状态”。对于ReentrantLock来说，state是线程获取锁的次数；对于CountDownLatch来说，则表示计数值的大小。

### 阻塞线程

接着来看await方法，直接调用了AQS的acquireSharedInterruptibly。

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

```java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

首先尝试获取共享锁，实现方式和独占锁类似，由CountDownLatch实现判断逻辑。

```cpp
protected int tryAcquireShared(int acquires) {
   return (getState() == 0) ? 1 : -1;
}
```

返回1代表获取成功，返回-1代表获取失败。如果获取失败，需要调用doAcquireSharedInterruptibly：

```java
private void doReleaseShared() {
   for (;;) {
       Node h = head;
       if (h != null && h != tail) {
           int ws = h.waitStatus;
           if (ws == Node.SIGNAL) {
             //1
               if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                   continue;            // loop to recheck cases
               unparkSuccessor(h);
           }
           //2
           else if (ws == 0 &&
                    !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
               continue;                // loop on failed CAS
       }
       if (h == head)                   // loop if head changed
           break;
   }
}
```

标记1里，头节点状态如果SIGNAL，则状态重置为0，并调用unparkSuccessor唤醒下个节点。

标记2里，被唤醒的节点状态会重置成0，在下一次循环中被设置成PROPAGATE状态，代表状态要向后传播

```java
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);

```

在唤醒线程的操作里，分成三步：

- 处理当前节点：非CANCELLED状态重置为0；
- 寻找下个节点：如果是CANCELLED状态，说明节点中途溜了，从队列尾开始寻找排在最前还在等着的节点
- 唤醒：利用LockSupport.unpark唤醒下个节点里的线程。

线程是在doAcquireSharedInterruptibly里被阻塞的，唤醒后调用到setHeadAndPropagate。

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    setHead(node);
    
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

setHead设置头节点后，再判断一堆条件，取出下一个节点，如果也是共享类型，进行doReleaseShared释放操作。下个节点被唤醒后，重复上面的步骤，达到共享状态向后传播。

要注意，await操作看着好像是独占操作，但它可以在多个线程中调用。当计数值等于0的时候，调用await的线程都需要知道，所以使用共享锁。

### 限定时间的await

CountDownLatch的await方法还有个限定阻塞时间的版本.



```java
public boolean await(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

跟踪代码，最后来看doAcquireSharedNanos方法，和上文介绍的doAcquireShared逻辑基本一样，不同之处是加了time字眼的处理。

```java
private boolean doAcquireSharedNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

进入方法时，算出能够执行多久的deadline，然后在循环中判断时间。注意到代码中间有句：

```undefined
nanosTimeout > spinForTimeoutThreshold
```

```java
static final long spinForTimeoutThreshold = 1000L;
```

spinForTimeoutThreshold写死了1000ns，这就是所谓的自旋操作。当超时在1000ns内，让线程在循环中自旋，否则阻塞线程。

### 总结

两篇文章分别以ReentrantLock和CountDownLatch为例研究了AQS的独占功能和共享功能。AQS里的主要方法研究得七七八八了，趁热打铁下一篇将会研究其他同步工具类的实现。

