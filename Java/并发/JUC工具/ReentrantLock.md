## ReentrantLock 和 公平锁

### 1. ReentrantLock 介绍

鉴于上一篇小傅哥已经基于，HotSpot虚拟机源码分析 [synchronized](https://bugstack.cn/interview/2020/10/28/面经手册-第15篇-码农会锁-synchronized-解毒-剖析源码深度分析.html) 实现和相应核心知识点，本来想在本章直接介绍下 ReentrantLock。但因为 ReentrantLock 知识点较多，因此会分几篇分别讲解，突出每一篇重点，避免猪八戒吞枣。

**介绍**：ReentrantLock 是一个可重入且独占式锁，具有与 synchronized 监视器(monitor enter、monitor exit)锁基本相同的行为和语意。但与 synchronized 相比，它更加灵活、强大、增加了轮询、超时、中断等高级功能以及可以创建公平和非公平锁。

### 2. ReentrantLock 知识链条

![image-20201113162548598](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201113162548598.png)

ReentrantLock 是基于 Lock 实现的可重入锁，所有的 Lock 都是基于 AQS 实现的，AQS 和 Condition 各自维护不同的对象，在使用 Lock 和 Condition 时，其实就是两个队列的互相移动。它所提供的共享锁、互斥锁都是基于对 state 的操作。而它的可重入是因为实现了同步器 Sync，在 Sync 的两个实现类中，包括了公平锁和非公平锁。

这个公平锁的具体实现，就是我们本章节要介绍的重点，了解什么是公平锁、公平锁的具体实现。*学习完基础的知识可以更好的理解 ReentrantLock*

### 3. ReentrantLock 公平锁代码

#### 3.1 初始化

```java
ReentrantLock lock = new ReentrantLock(true);  // true：公平锁
lock.lock();
try {
    // todo
} finally {
    lock.unlock();
}
```

- 初始化构造函数入参，选择是否为初始化公平锁。
- 其实一般情况下并不需要公平锁，除非你的场景中需要保证顺序性。
- 使用 ReentrantLock 切记需要在 finally 中关闭，`lock.unlock()`。

#### 3.2 公平锁、非公平锁，选择

```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

- 构造函数中选择公平锁（FairSync）、非公平锁（NonfairSync）。

#### 3.3 hasQueuedPredecessors

```java
static final class FairSync extends Sync {

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
		...
    }
}
```

- 公平锁和非公平锁，主要是在方法 `tryAcquire` 中，是否有 `!hasQueuedPredecessors()` 判断。

#### 3.4 队列首位判断

```java
public final boolean hasQueuedPredecessors() {
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

- 在这个判断中主要就是看当前线程是不是同步队列的首位，是：true、否：false
- 这部分就涉及到了公平锁的实现，CLH（Craig，Landin andHagersten）。*三个作者的首字母组合*

## 四、什么是公平锁

#### 1.1 看图说话

![图 16-3 CLH 实现过程原理图](https://bugstack.cn/assets/images/2020/interview/interview-16-03.png)

#### 1.2 代码实现

```java
public class CLHLock implements Lock {

    private final ThreadLocal<CLHLock.Node> prev;
    private final ThreadLocal<CLHLock.Node> node;
    private final AtomicReference<CLHLock.Node> tail = new AtomicReference<>(new CLHLock.Node());

    private static class Node {
        private volatile boolean locked;
    }

    public CLHLock() {
        this.prev = ThreadLocal.withInitial(() -> null);
        this.node = ThreadLocal.withInitial(CLHLock.Node::new);
    }

    @Override
    public void lock() {
        final Node node = this.node.get();
        node.locked = true;
        Node pred_node = this.tail.getAndSet(node);
        this.prev.set(pred_node);
        // 自旋
        while (pred_node.locked);
    }

    @Override
    public void unlock() {
        final Node node = this.node.get();
        node.locked = false;
        this.node.set(this.prev.get());
    }

}
```

#### 1.3 代码讲解

CLH（Craig，Landin and Hagersten），是一种基于链表的可扩展、高性能、公平的自旋锁。

在这段代码的实现过程中，相当于是虚拟出来一个链表结构，由 AtomicReference 的方法 getAndSet 进行衔接。*getAndSet 获取当前元素，设置新的元素*

**lock()**

- 通过 `this.node.get()` 获取当前节点，并设置 locked 为 true。
- 接着调用 `this.tail.getAndSet(node)`，获取当前尾部节点 pred_node，同时把新加入的节点设置成尾部节点。
- 之后就是把 this.prev 设置为之前的尾部节点，也就相当于链路的指向。
- 最后就是自旋 `while (pred_node.locked)`，直至程序释放。

**unlock()**

- 释放锁的过程就是拆链，把释放锁的节点设置为false `node.locked = false`。
- 之后最重要的是把当前节点设置为上一个节点，这样就相当于把自己的节点拆下来了，等着垃圾回收。

`CLH`队列锁的优点是空间复杂度低，在SMP（Symmetric Multi-Processor）对称多处理器结构（一台计算机由多个CPU组成，并共享内存和其他资源，所有的CPU都可以平等地访问内存、I/O和外部中断）效果还是不错的。但在 NUMA(Non-Uniform Memory Access) 下效果就不太好了，这部分知识可以自行扩展。

### 2. MCSLock

#### 2.1 代码实现

```java
public class MCSLock implements Lock {

    private AtomicReference<MCSLock.Node> tail = new AtomicReference<>(null);
    ;
    private ThreadLocal<MCSLock.Node> node;

    private static class Node {
        private volatile boolean locked = false;
        private volatile Node next = null;
    }

    public MCSLock() {
        node = ThreadLocal.withInitial(Node::new);
    }

    @Override
    public void lock() {
        Node node = this.node.get();
        Node preNode = tail.getAndSet(node);
        if (null == preNode) {
            node.locked = true;
            return;
        }
        node.locked = false;
        preNode.next = node;
        while (!node.locked) ;
    }

    @Override
    public void unlock() {
        Node node = this.node.get();
        if (null != node.next) {
            node.next.locked = true;
            node.next = null;
            return;
        }
        if (tail.compareAndSet(node, null)) {
            return;
        }
        while (node.next == null) ;
    }

}
```

#### 2.1 代码讲解

MCS 来自于发明人名字的首字母： John Mellor-Crummey和Michael Scott。

它也是一种基于链表的可扩展、高性能、公平的自旋锁，但与 CLH 不同。它是真的有下一个节点 next，添加这个真实节点后，它就可以只在本地变量上自旋，而 CLH 是前驱节点的属性上自旋。

因为自旋节点的不同，导致 CLH 更适合于 SMP 架构、MCS 可以适合 NUMA 非一致存储访问架构。你可以想象成 CLH 更需要线程数据在同一块内存上效果才更好，MCS 因为是在本店变量自选，所以无论数据是否分散在不同的CPU模块都没有影响。

代码实现上与 CLH 没有太多差异，这里就不在叙述了，可以看代码学习。