## STACK

我们说`Stack`栈，这个实现类已经不推荐使用了，需要从它的源码上看。

```java
/**
 *
 * <p>A more complete and consistent set of LIFO stack operations is
 * provided by the {@link Deque} interface and its implementations, which
 * should be used in preference to this class.  For example:
 * <pre>   {@code
 *   Deque<Integer> stack = new ArrayDeque<Integer>();}</pre>
 *   
 * @author  Jonathan Payne
 * @since   JDK1.0
 */
public class Stack<E> extends Vector<E> 
s.push("aaa");

public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

1. `Stack` 栈是在JDK1.0时代时，基于继承`Vector`，实现的。本身`Vector`就是一个不推荐使用的类，主要在于它的一些操作方法锁🔒(*synchronized*)的力度太粗，都是放到方法上。
2. `Stack` 栈底层是使用`Vector`数组实现，在学习`ArrayList`时候我们知道，数组结构在元素添加和擅长需要通过`System.arraycopy`，进行扩容操作。而本身栈的特点是首尾元素的操作，也不需要遍历，使用数组结构其实并不太理想。
3. 同时在这个方法的注释上也明确标出来，推荐使用`Deque<Integer> stack = new ArrayDeque<Integer>();`，虽然这也是数组结构，但是它没有粗粒度的锁，同时可以申请指定空间并且在扩容时操作时也要优于`Stack` 。并且它还是一个双端队列，使用起来更灵活。



## 双端队列ArrayDeque

`ArrayDeque` 是基于数组实现的可动态扩容的双端队列，也就是说你可以在队列的头和尾同时插入和弹出元素。当元素数量超过数组初始化长度时，则需要扩容和迁移数据。

**数据结构和操作**，如下；

![小傅哥 bugstack.cn & 双端队列数据结构操作](https://bugstack.cn/assets/images/2020/interview/interview-10-04.png)

从上图我们可以了解到如下几个知识点；

1. 双端队列是基于数组实现，所以扩容迁移数据操作。
2. `push`，像结尾插入、`offerLast`，向头部插入，这样两端都满足后进先出。
3. 整体来看，双端队列，就是一个环形。所以扩容后继续插入元素也满足后进先出。

#### 2.1 功能使用

```java
@Test
public void test_ArrayDeque() {
    Deque<String> deque = new ArrayDeque<String>(1);
    
    deque.push("a");
    deque.push("b");
    deque.push("c");
    deque.push("d");
    
    deque.offerLast("e");
    deque.offerLast("f");
    deque.offerLast("g");
    deque.offerLast("h");  // 这时候扩容了
    
    deque.push("i");
    deque.offerLast("j");
    
    System.out.println("数据出栈：");
    while (!deque.isEmpty()) {
        System.out.print(deque.pop() + " ");
    }
}
```

以上这部分代码就是与上图的展现是一致的，按照图中的分析我们看下输出结果，如下；

```
数据出栈：
i d c b a e f g h j 
Process finished with exit code 0
```

- `i d c b a e f g h j `，正好满足了我们的说的数据出栈顺序。*可以参考上图再进行理解*

#### 2.2 源码分析

`ArrayDeque` 这种双端队列是基于数组实现的，所以源码上从初始化到数据入栈扩容，都会有数组操作的痕迹。接下来我们就依次分析下。

##### 2.2.1 初始化

`new ArrayDeque<String>(1);`，其实它的构造函数初始化默认也提供了几个方法，比如你可以指定大小以及提供默认元素。

```java
private static int calculateSize(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;
        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 element
    }
    return initialCapacity;
}
```

- 在初始化的过程中，它需要找到你当前传输值最小的2的倍数的一个容量。这与HashMap的初始化过程相似。

##### 2.2.2 数据入栈

**addFirst是从后往前添加 8->0，addLast是从前往后添加 0->8，first本质是添加到栈低，addlast本质是添加到栈顶**

`deque.push("a");`，ArrayDeque，提供了一个 push 方法，这个方法与`deque.offerFirst(“a”)`，一致，因为它们的底层源码是一样的，如下；

**addFirst：**

```java
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
  	//因为我们的数组长度是2^n的倍数，所以 2^n - 1 就是一个全是1的二进制数，
    // 可以用于与运算得出数组下标。
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}
```

**addLast：**

```java
public void addLast(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[tail] = e;
    if ( (tail = (tail + 1) & (elements.length - 1)) == head)
        doubleCapacity();
}
```

这部分入栈元素，其实就是给数组赋值，知识点如下；

1. 在`addFirst()`中，定位下标，`head = (head - 1) & (elements.length - 1)`，因为我们的数组长度是`2^n`的倍数，所以 `2^n - 1` 就是一个全是1的二进制数，可以用于与运算得出数组下标。
2. 同样`addLast()`中，也使用了相同的方式定位下标，只不过它是从0开始，往上增加。
3. 最后，当头(head)与尾(tile)，数组则需要两倍扩容`doubleCapacity`。

下标计算：`head = (head - 1) & (elements.length - 1)`：

- (0 - 1) & (8 - 1) = 7
- (7 - 1) & (8 - 1) = 6
- (6 - 1) & (8 - 1) = 5
- …

##### 2.2.3 两倍扩容，数据迁移

```java
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0;
    tail = n;
}
```

其实以上这部分源码，就是进行两倍*n « 1*扩容，同时把两端数据迁移进新的数组，整个操作过程也与我们上图对应。为了更好的理解，我们单独把这部分代码做一些测试。

**测试代码：**

```java
@Test
public void test_arraycopy() {
    int head = 0, tail = 0;
    Object[] elements = new Object[8];
    elements[head = (head - 1) & (elements.length - 1)] = "a";
    elements[head = (head - 1) & (elements.length - 1)] = "b";
    elements[head = (head - 1) & (elements.length - 1)] = "c";
    elements[head = (head - 1) & (elements.length - 1)] = "d";
    
    elements[tail] = "e";
    tail = (tail + 1) & (elements.length - 1);
    elements[tail] = "f";
    tail = (tail + 1) & (elements.length - 1);
    elements[tail] = "g";
    tail = (tail + 1) & (elements.length - 1);
    elements[tail] = "h";
    tail = (tail + 1) & (elements.length - 1);
    
    System.out.println("head：" + head);
    System.out.println("tail：" + tail);
    
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    
    // 输出当前的元素
    System.out.println(JSON.toJSONString(elements));
    
    // head == tail 扩容
    Object[] a = new Object[8 << 1];
    System.arraycopy(elements, p, a, 0, r);
    System.out.println(JSON.toJSONString(a));
    System.arraycopy(elements, 0, a, r, p);
    System.out.println(JSON.toJSONString(a));
    
    elements = a;
    head = 0;
    tail = n;
    a[head = (head - 1) & (a.length - 1)] = "i";
    System.out.println(JSON.toJSONString(a));
}
```

以上的测试过程主要模拟了8个长度的空间的数组，在进行双端队列操作时数组扩容，数据迁移操作，可以单独运行，测试结果如下；

```
head：4
tail：4
["e","f","g","h","d","c","b","a"]
["d","c","b","a",null,null,null,null,null,null,null,null,null,null,null,null]
["d","c","b","a","e","f","g","h",null,null,null,null,null,null,null,null]
["d","c","b","a","e","f","g","h","j",null,null,null,null,null,null,"i"]

Process finished with exit code 0
```

从测试结果可以看到；

1. 当head与tail相等时，进行扩容操作。
2. 第一次数据迁移，`System.arraycopy(elements, p, a, 0, r);`，**d、c、b、a**，落入新数组。
3. 第二次数据迁移，`System.arraycopy(elements, 0, a, r, p);`，**e、f、g、h**，落入新数组。
4. 最后再尝试添加新的元素，i和j。每一次的输出结果都可以看到整个双端链路的变化。



## 3. 双端队列LinkedList

`Linkedlist`天生就可以支持双端队列，而且从头尾取数据也是它时间复杂度O(1)的。同时数据的插入和删除也不需要像数组队列那样拷贝数据，虽然`Linkedlist`有这些优点，但不能说`ArrayDeque`因为有数组复制性能比它低。

**Linkedlist，数据结构**：

![img](https://bugstack.cn/assets/images/2020/interview/interview-10-05.png)

#### 3.1 功能使用

```java
@Test
public void test_Deque_LinkedList(){
    Deque<String> deque = new LinkedList<>();
    deque.push("a");
    deque.push("b");
    deque.push("c");
    deque.push("d");
    deque.offerLast("e");
    deque.offerLast("f");
    deque.offerLast("g");
    deque.offerLast("h"); 
    deque.push("i");
    deque.offerLast("j");
    
    System.out.println("数据出栈：");
    while (!deque.isEmpty()) {
        System.out.print(deque.pop() + " ");
    }
}
```

**测试结果**：

```
数据出栈：
i d c b a e f g h j 

Process finished with exit code 0
```

- 测试结果上看与使用`ArrayDeque`是一样的，功能上没有差异。

#### 3.2 源码分析

**压栈**：`deque.push("a");`、`deque.offerFirst("a");`

```java
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

**压栈**：`deque.offerLast("e");`

```java
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

- `linkFirst`、`linkLast`，两个方法分别是给链表的首尾节点插入元素，因为这是链表结构，所以也不存在扩容，只需要把双向链路链接上即可。

## 4. 延时队列DelayQueue

```
你是否有时候需要把一些数据存起来，倒计时到某个时刻在使用？
```

在Java的队列数据结构中，还有一种队列是延时队列，可以通过设定存放时间，依次轮训获取。

#### 4.1 功能使用

**先写一个Delayed的实现类**：

```java
public class TestDelayed implements Delayed {

    private String str;
    private long time;

    public TestDelayed(String str, long time, TimeUnit unit) {
        this.str = str;
        this.time = System.currentTimeMillis() + (time > 0 ? unit.toMillis(time) : 0);
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return time - System.currentTimeMillis();
    }

    @Override
    public int compareTo(Delayed o) {
        TestDelayed work = (TestDelayed) o;
        long diff = this.time - work.time;
        if (diff <= 0) {
            return -1;
        } else {
            return 1;
        }
    }

    public String getStr() {
        return str;
    }
}
```

- 这个相当于延时队列的一个固定模版方法，通过这种方式来控制延时。

**案例测试**：

```java
@Test
public void test_DelayQueue() throws InterruptedException {
    DelayQueue<TestDelayed> delayQueue = new DelayQueue<TestDelayed>();
    delayQueue.offer(new TestDelayed("aaa", 5, TimeUnit.SECONDS));
    delayQueue.offer(new TestDelayed("ccc", 1, TimeUnit.SECONDS));
    delayQueue.offer(new TestDelayed("bbb", 3, TimeUnit.SECONDS));
    
    logger.info(((TestDelayed) delayQueue.take()).getStr());
    logger.info(((TestDelayed) delayQueue.take()).getStr());
    logger.info(((TestDelayed) delayQueue.take()).getStr());
}
```

**测试结果**：

```
01:44:21.000 [main] INFO  org.itstack.interview.test.ApiTest - ccc
01:44:22.997 [main] INFO  org.itstack.interview.test.ApiTest - bbb
01:44:24.997 [main] INFO  org.itstack.interview.test.ApiTest - aaa

Process finished with exit code 0
```

- 在案例测试中我们分别设定不同的休眠时间，1、3、5，TimeUnit.SECONDS。
- 测试结果分别在21、22、24，输出了我们要的队列结果。
- 队列中的元素不会因为存放的先后顺序而导致输出顺序，它们是依赖于休眠时长决定。

#### 4.2 源码分析

##### 4.2.1 元素入栈

**入栈：**：`delayQueue.offer(new TestDelayed("aaa", 5, TimeUnit.SECONDS));`

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}

private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

- 关于数据存放还有 `ReentrantLock` 可重入锁🔒，但暂时不是我们本章节数据结构的重点，后面章节会介绍到。
- `DelayQueue` 是基于数组实现的，所以可以动态扩容，另外它插入元素的顺序并不影响最终的输出顺序。
- 而元素的排序依赖于compareTo方法进行排序，也就是休眠的时间长短决定的。
- 同时只有实现了`Delayed`接口，才能存放元素。

##### 4.2.2 元素出栈

**出栈**：`delayQueue.take()`

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS
                if (delay <= 0)
                    return q.poll();
                first = null; // don't retain ref while
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentT
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```

- 这部分的代码有点长，主要是元素的获取。`DelayQueue` 是 `Leader-Followr` 模式的变种，消费者线程处于等待await时，总是等待最先休眠完成的元素。
- 这里会最小化的空等时间，提高线程利用率。*数据结构讲完后，后面会有专门章节介绍*

# 【9.8】SynchronousQueue



无缓冲阻塞队列，用来在两个线程之间移交元素

模式相同则入栈（队），不同则出栈（队），所以并非真正的无缓冲

队列为空也入栈（队）

并不是真正的队列，不维护存储空间，维护的是一组线程，这些线程在等待着放入或者移出元素

这种阻塞队列确实是非常复杂的，但是却非常有用。SynchronousQueue是一种极为特殊的阻塞队列，它没有实际的容量，任意线程（生产者线程或者消费者线程，生产类型的操作比如put，offer，消费类型的操作比如poll，take）都会等待知道获得数据或者交付完成数据才会返回，一个生产者线程的使命是将线程附着着的数据交付给一个消费者线程，而一个消费者线程则是等待一个生产者线程的数据。它们会在匹配到互斥线程的时候就会做数据交易，比如生产者线程遇到消费者线程时，或者消费者线程遇到生产者线程时，一个生产者线程就会将数据交付给消费者线程，然后共同退出。在java线程池newCachedThreadPool中就使用了这种阻塞队列。

### 优点

将更多关于任务状态的信息反馈给生产者。当交付被接受时，它就知道消费者已经得到了任务，而不是简单地把任务放入一个队列——这种区别就好比将文件直接交给同事，还是将文件放到她的邮箱中并希望她能尽快拿到文件。

### 成员变量

```java
// CPU的数量
static final int NCPUS = Runtime.getRuntime().availableProcessors();
// 有超时的情况自旋多少次，当CPU数量小于2的时候不自旋
static final int maxTimedSpins = (NCPUS < 2) ? 0 : 32;
// 没有超时的情况自旋多少次
static final int maxUntimedSpins = maxTimedSpins * 16;
// 针对有超时的情况，自旋了多少次后，如果剩余时间大于1000纳秒就使用带时间的LockSupport.parkNanos()这个方法
static final long spinForTimeoutThreshold = 1000L;
// 传输器，即两个线程交换元素使用的东西
private transient volatile Transferer<E> transferer;
//主要定义了一个transfer方法用来传输元素
abstract static class Transferer<E> {
    abstract E transfer(E e, boolean timed, long nanos);
}
// 以栈方式实现的Transferer
static final class TransferStack<E> extends Transferer<E> {
    // 栈中节点的几种类型： 
    static final int REQUEST    = 0;// 1. 消费者（请求数据的）
    static final int DATA       = 1;// 2. 生产者（提供数据的）
    static final int FULFILLING = 2;// 3. 二者正在匹配中
    // 栈中的节点
    static final class SNode {
        volatile SNode next;        // 下一个节点
        volatile SNode match;      // 匹配者     
        volatile Thread waiter;     // 等待着的线程      
        Object item;                // 元素  
        int mode;//也就是节点的类型，是消费者，是生产者，还是正在匹配中
    }
    volatile SNode head;// 栈的头节点
}
// 以队列方式实现的Transferer
static final class TransferQueue<E> extends Transferer<E> {
    // 队列中的节点
    static final class QNode {
        volatile QNode next;          // 下一个节点
        volatile Object item;         // 存储的元素   
        volatile Thread waiter;       // 等待着的线程
        final boolean isData;// 是否是数据节点
    }
    transient volatile QNode head;// 队列的头节点
    transient volatile QNode tail;// 队列的尾节点
}
```

### 构造器

```java
public SynchronousQueue() {
    this(false);// 默认非公平模式
}
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();// 公平模式使用队列，非公平模式使用栈
}
```

### 入队

```java
public void put(E e) throws InterruptedException {  
    if (e == null) throw new NullPointerException();// 元素不可为null
    // 三个参数分别是：传输的元素，是否需要超时，超时的时间
    if (transferer.transfer(e, false, 0) == null) {
        // 如果传输失败，直接让线程中断并抛出中断异常
        Thread.interrupted();
        throw new InterruptedException();
    }
}
```

### 出队

```java
public E take() throws InterruptedException {
    // 第一个参数为null表示是消费者，要取元素
    E e = transferer.transfer(null, false, 0);
    if (e != null)// 如果取到了元素就返回
        return e;
    // 否则让线程中断并抛出中断异常
    Thread.interrupted();
    throw new InterruptedException();
}
```

### 栈的transfer

```java
E transfer(E e, boolean timed, long nanos) {
    SNode s = null; 
    int mode = (e == null) ? REQUEST : DATA;// 根据e是否为null决定是生产者还是消费者
    for (;;) {// 自旋
        SNode h = head;// 栈顶元素 
        if (h == null || h.mode == mode) {// 入栈
            if (timed && nanos <= 0) {     // 如果有超时设置而且已到期，不能再入栈,协助清理cancel状态的元素
                if (h != null && h.isCancelled())// 如果头节点不为空且是取消状态
                    casHead(h, h.next);//头节点弹出，将h.next设置为新的head，并进入下一次循环
                else  
                    return null;// 否则，直接返回null（超时返回null）
            } else if (casHead(h, s = snode(s, e, h, mode))) {// 入栈成功      
                // 调用awaitFulfill()方法自旋+阻塞当前入栈的线程并等待被匹配到
                SNode m = awaitFulfill(s, timed, nanos);
                // 如果m等于s，说明取消了，那么就把它清除掉，并返回null
                if (m == s) {               
                    clean(s);
                    return null;// 被取消了返回null
                }
                
                // 到这里说明匹配到元素了,因为从awaitFulfill()里面出来要不被取消了要不就匹配到了,如果头节点不为空，并且头节点的下一个节点是s,就把头节点换成s的下一个节点,也就是把h和s都弹出了,也就是把栈顶两个元素都弹出了
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     
                // 根据当前节点的模式判断返回m还是s中的值
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        } else if (!isFulfilling(h.mode)) {     
            if (h.isCancelled())// 节点和当前节点模式不一样,如果头节点不是正在匹配中并且已经取消了，就把它弹出栈         
                casHead(h, h.next);         
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                // 头节点没有在匹配中，就让当前节点先入队，再让他们尝试匹配
                // 且s成为了新的头节点，它的状态是正在匹配中
                for (;;) { 
                    SNode m = s.next;       
                    // 如果m为null，说明除了s节点外的节点都被其它线程先一步匹配掉了
                    // 就清空栈并跳出内部循环，到外部循环再重新入栈判断
                    if (m == null) {        
                        casHead(s, null);   
                        s = null;           
                        break;            
                    }
                    SNode mn = m.next;
                    // 如果m和s尝试匹配成功，就弹出栈顶的两个元素m和s
                    if (m.tryMatch(s)) {
                        casHead(s, mn);     
                        // 返回匹配结果
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                 
                        // 尝试匹配失败，说明m已经先一步被其它线程匹配了,就协助清除它
                        s.casNext(m, mn);   
                }
            }
        } else {                            
            //当前节点和头节点模式不一样,且头节点是正在匹配中
            SNode m = h.next;              
            if (m == null)                 
                // 如果m为null，说明m已经被其它线程先一步匹配了
                casHead(h, null);           
            else {
                SNode mn = m.next;
                // 协助匹配，如果m和s尝试匹配成功，就弹出栈顶的两个元素m和s
                if (m.tryMatch(h))          
                    // 将栈顶的两个元素弹出后，再让s重新入栈
                    casHead(h, mn);         
                else                        
                    // 尝试匹配失败，说明m已经先一步被其它线程匹配了
                    // 就协助清除它
                    h.casNext(m, mn);       
            }
        }
    }
}
// 三个参数：需要等待的节点，是否需要超时，超时时间
//等待其他的线程来匹配，这个线程一直阻塞直到被匹配，在阻塞之前首先会自旋，这个自旋会在阻塞之前进行，它会调用shouldSpin方法来进行判断是否需要自选
SNode awaitFulfill(SNode s, boolean timed, long nanos) {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;// 到期时间
    Thread w = Thread.currentThread();// 当前线程
    int spins = (shouldSpin(s) ? (timed ? maxTimedSpins : maxUntimedSpins) : 0);    // 自旋次数
    for (;;) {
        if (w.isInterrupted())// 当前线程中断了，尝试清除s
            s.tryCancel();
        
        // 检查s是否匹配到了元素m（有可能是其它线程的m匹配到当前线程的s）
        SNode m = s.match;
        if (m != null)// 如果匹配到了，直接返回m
            return m;
        
        // 如果需要超时
        if (timed) {
            // 检查超时时间如果小于0了，尝试清除s
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                s.tryCancel();
                continue;
            }
        }
        if (spins > 0)
            // 如果还有自旋次数，自旋次数减一，并进入下一次自旋
            spins = shouldSpin(s) ? (spins-1) : 0;
        
        // 后面的elseif都是自旋次数没有了
        else if (s.waiter == null)
            // 如果s的waiter为null，把当前线程注入进去，并进入下一次自旋
            s.waiter = w; // establish waiter so can park next iter
        else if (!timed)
            // 如果不允许超时，直接阻塞，并等待被其它线程唤醒，唤醒后继续自旋并查看是否匹配到了元素
            LockSupport.park(this);
        else if (nanos > spinForTimeoutThreshold)
            // 如果允许超时且还有剩余时间，就阻塞相应时间
            LockSupport.parkNanos(this, nanos);
    }
}
// SNode里面的方向，调用者m是s的下一个节点
// 这时候m节点的线程应该是阻塞状态的
boolean tryMatch(SNode s) {
    // 如果m还没有匹配者，就把s作为它的匹配者
    if (match == null &&
        UNSAFE.compareAndSwapObject(this, matchOffset, null, s)) {
        Thread w = waiter;
        if (w != null) {    
            waiter = null;
            // 唤醒m中的线程，两者匹配完毕
            LockSupport.unpark(w);
        }
        // 匹配到了返回true
        return true;
    }
    // 可能其它线程先一步匹配了m，返回其是否是s
    return match == s;
}
```

如果当前的交易栈是空的，或者包含与请求交易节点模式相同的节点，那么就将这个请求交易的节点作为新的栈顶节点，等待被下一个请求交易的节点匹配，最后会返回匹配节点的数据或者null，如果被取消则会返回null。

如果当前交易栈不为空，并且请求交易的节点和当前栈顶节点模式互补，那么将这个请求交易的节点的模式变为FULFILLING，然后将其压入栈中，和互补的节点进行匹配，完成交易之后将两个节点一起弹出，并且返回交易的数据。

如果栈顶已经存在一个模式为FULFILLING的节点，说明栈顶的节点正在进行匹配，那么就帮助这个栈顶节点快速完成交易，然后继续交易。

### 队列的transfer

```java
E transfer(E e, boolean timed, long nanos) {  
    
 //在每一种情况，执行的过程中，检查和尝试帮助其他stalled/slow线程移动队列头和尾节点 循环开始，首先进行null检查，防止未初始队列头和尾节点。当然这种情况，在当前同步队列中，不可能发生，如果调用持有transferer的non-volatile/final引用， 可能出现这种情况。一般在循环的开始，都要进行null检查，检查过程非常快，不用过多担心性能问题。 
         
    QNode s = null; 
    //如果元素e不为null，则为DATA模式，否则为REQUEST模式  
    boolean isData = (e != null);  
    for (;;) {  
        QNode t = tail;  
        QNode h = head;  
        //如果队列头或尾节点没有初始化，则自旋  
        if (t == null || h == null)           
            continue;                       
        if (h == t || t.isData == isData) { //如果队列为空，或当前节点与队尾模式相同 ,入队 
            QNode tn = t.next;  
            if (t != tail)                  //如果t不是队尾，非一致性读取，自旋 
                continue;  
            if (tn != null) {               //tn不为null，说明有其他线程添加了tn结点  （设置了tail.next）           
                advanceTail(t, tn);  //如果t.next不为null，设置新的队尾，自旋  
                continue;  
            }  
            if (timed && nanos <= 0) //如果超时，且超时时间小于0，则返回null  
                return null;  
            if (s == null)  
                s = new QNode(e, isData);  //根据元素和模式构造节点
            if (!t.casNext(null, s))        // 新节点入队列失败(t.next被赋值了)，自旋
                continue;
            //设置队尾为当前节点  
            advanceTail(t, s);              // swing tail and wait  
            //自旋或阻塞直到节点被fulfilled  
            Object x = awaitFulfill(s, e, timed, nanos);  
            if (x == s) {                   // wait was cancelled  
                //如果s指向自己，s出队列，并清除队列中取消等待的线程节点  
                clean(t, s);  
                return null;  
            }  
            if (!s.isOffList()) {           // s仍然在队列中 
                advanceHead(t, s);          
                if (x != null)              
                    s.item = s;  
                s.waiter = null;  
            }  
            //如果自旋等待匹配的节点元素不为null，则返回x，否则返回e  
            return (x != null) ? x : e;  
        } else {                              
            //如果队列不为空，且与队头的模式不同，及匹配成功 （与队尾匹配成功，则一定与队头匹配成功！） 
            QNode m = h.next;                
            if (t != tail || m == null || h != head)  
                //如果h不为当前队头，则返回，即读取不一致  
                continue;                   
            Object x = m.item;  
            if (
                isData == (x != null) ||   
                x == m ||                    
                !m.casItem(x, e)   
            ){        
                
                advanceHead(h, m);          //如果队头后继，取消等待，则出队列  
                continue;  
            }  
            //否则匹配成功  
            advanceHead(h, m);                
            //unpark等待线程  
            LockSupport.unpark(m.waiter);  
            //如果匹配节点元素不为null，则返回x，否则返回e，即take操作，返回等待put线程节点元素，  
            //put操作，返回put元素  
            return (x != null) ? x : e;  
        }  
    }  
}
```

如果队列为空，或者请求交易的节点和队列中的节点具有相同的交易类型，那么就将该请求交易的节点添加到队列尾部等待交易，直到被匹配或者被取消

如果队列中包含了等待的节点，并且请求的节点和等待的节点是互补的，那么进行匹配并且进行交易

SynchronousQueue一般用于生产、消费的速度大致相当的情况，这样才不会导致系统中过多的线程处于阻塞状态。



# 【9.8】ArrayBlockingQueue



```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
```

BlockingQueue：定义了队列的入队出队的方法

AbstractQueue：入队出队的基本操作

### 成员变量

组成：一个对象数组+1把锁ReentrantLock+2个条件Condition

```java
// 使用数组存储元素
final Object[] items;
// 取元素的指针  记录下一次操作的位置
int takeIndex;
// 放元素的指针   记录下一次操作的位置
int putIndex;
// 元素数量
int count;
// 保证并发访问的锁
final ReentrantLock lock;
//等待出队的条件 消费者监视器
private final Condition notEmpty;
//等待入队的条件   生产者监视器
private final Condition notFull;
```

### 构造器

```java
//必须传入容量，可以控制重入锁是公平还是非公平
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    // 初始化数组
    this.items = new Object[capacity];
    // 创建重入锁及两个条件
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
public ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c) {
    this(capacity, fair);//final修饰的变量不会发生指令重排
    final ReentrantLock lock = this.lock;
    lock.lock(); // 保证可见性 不是为了互斥  防止指令重排 保证item的安全
    try {
        int i = 0;
        try {
            for (E e : c) {
                checkNotNull(e);
                items[i++] = e;
            }
        } catch (ArrayIndexOutOfBoundsException ex) {
            throw new IllegalArgumentException();
        }
        count = i;
        putIndex = (i == capacity) ? 0 : i;
    } finally {
        lock.unlock();
    }
}
```

对象创建三步：2和3可能会发生重排（互相不依赖）

1、分配内存空间；

2、初始化对象；

3、将内存空间的地址赋值给对应的引用。

### 入队

```java
public boolean add(E e) {
    return super.add(e);
}
// AbstractQueue 调用offer(e)如果成功返回true，如果失败抛出异常
public boolean add(E e) {   
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
public boolean offer(E e) {    
    checkNotNull(e);// 元素不可为空
    final ReentrantLock lock = this.lock;    
    lock.lock();// 加锁
    try {
        if (count == items.length)// 如果数组满了就返回false            
            return false;
        else {            
            enqueue(e);// 如果数组没满就调用入队方法并返回true
            return true;
        }
    } finally {        
        lock.unlock();
    }
}
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;    
    lock.lockInterruptibly();// 加锁，如果线程中断了抛出异常
    try {             
        // 这里之所以使用while而不是if,是因为有可能多个线程阻塞在lock上,即使唤醒了可能其它线程先一步修改了队列又变成满的了,因此必须重新判断，再次等待
        while (count == items.length)// 如果数组满了，使用notFull等待
            notFull.await();// notFull等待表示现在队列满了,等待被唤醒(只有取走一个元素后，队列才不满)        
        enqueue(e);// 入队
    } finally {
        lock.unlock();
    }
}
public boolean offer(E e, long timeout, TimeUnit unit)throws InterruptedException {
    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        // 如果数组满了，就阻塞nanos纳秒，如果唤醒这个线程时依然没有空间且时间到了就返回false
        while (count == items.length) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(e);//入队
        return true;
    } finally {
        lock.unlock();
    }
}
private void enqueue(E x) {
    final Object[] items = this.items;    
    items[putIndex] = x;// 把元素直接放在放指针的位置上    
    if (++putIndex == items.length)// 如果放指针到数组尽头了，就返回头部
        putIndex = 0;  
    count++;// 数量加1  
    notEmpty.signal();// 唤醒notEmpty，因为入队了一个元素，所以肯定不为空了
}
```

- add(e)时如果队列满了则抛出异常；
- offer(e)时如果队列满了则返回false；
- put(e)时如果队列满了则使用notFull等待；
- offer(e, timeout, unit)时如果队列满了则等待一段时间后如果队列依然满就返回false；
- 利用放指针循环使用数组来存储元素；

### 出队

```java
public E remove() {    
    E x = poll();// 调用poll()方法出队
    if (x != null)// 如果有元素出队就返回这个元素     
        return x;
    else 
        throw new NoSuchElementException();// 如果没有元素出队就抛出异常
}
public E poll() {
    final ReentrantLock lock = this.lock;    
    lock.lock();
    try {       
        return (count == 0) ? null : dequeue();// 如果队列没有元素则返回null，否则出队
    } finally {
        lock.unlock();
    }
}
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;   
    lock.lockInterruptibly();
    try {  
        while (count == 0) // 如果队列无元素，则阻塞等待在条件notEmpty上
            notEmpty.await();
        return dequeue();// 有元素了再出队
    } finally {
        lock.unlock();
    }
}
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        // 如果队列无元素，则阻塞等待nanos纳秒
        // 如果下一次这个线程获得了锁但队列依然无元素且已超时就返回null
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
private E dequeue() {
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];// 取出指针位置的元素
    items[takeIndex] = null;// 把取指针位置设为null
    if (++takeIndex == items.length)// 将指针前移，如果数组到头了就返回数组前端循环利用
        takeIndex = 0;  
    count--;// 元素数量减1
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();// 唤醒notFull条件
    return x;
}
```

- remove()时如果队列为空则抛出异常；
- poll()时如果队列为空则返回null；
- take()时如果队列为空则阻塞等待在条件notEmpty上；
- poll(timeout, unit)时如果队列为空则阻塞等待一段时间后如果还为空就返回null；
- 利用取指针循环从数组中取元素；



### 指定元素删除

```java
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {  
        if (count > 0) {//如果此时队列不为null
            final int putIndex = this.putIndex;//获取下一个要添加元素时的索引      
            int i = takeIndex;//从出队索引位置开始查找     
            do {            
                if (o.equals(items[i])) {//查找要删除的元素，直接删除
                    removeAt(i);
                    return true;
                }
                
                if (++i == items.length)//若为true，说明已到数组尽头，将i设置为0,继续查找
                    i = 0; 
            } while (i != putIndex);//不等继续查找，如果相等，说明队列已经查找完毕
        }
        return false;
    } finally {
        lock.unlock();
    }
}

//把删除索引之后的元素往前移动一个位置
void removeAt(final int removeIndex) {
    final Object[] items = this.items;
    if (removeIndex == takeIndex) { //先判断要删除的元素是否为当前队列头元素,如果是直接删除
        items[takeIndex] = null;
        if (++takeIndex == items.length)//判断出队索引已经到达数组最末，出队索引设置为0
            takeIndex = 0;
        count--;//队列元素减1
        if (itrs != null)
            itrs.elementDequeued();//更新迭代器中的数据
    } else {
        //如果要删除的元素不在队列头部，
        //那么只需循环迭代把删除元素后面的所有元素往前移动一个位置
        //获取下一个要被添加的元素的索引，作为循环判断结束条件
        final int putIndex = this.putIndex;
        //执行循环
        for (int i = removeIndex;;) {
            //获取要删除节点索引的下一个索引
            int next = i + 1;
            //判断是否已为数组长度，如果是从数组头部（索引为0）开始找
            if (next == items.length)
                next = 0;
            //如果查找的索引不等于要添加元素的索引，说明元素可以再移动
            if (next != putIndex) {
                items[i] = items[next];//把后一个元素前移覆盖要删除的元
                i = next;
            } else {
                //在removeIndex索引之后的元素都往前移动完毕后清空最后一个元素
                items[i] = null;
                this.putIndex = i;
                break;//结束循环
            }
        }
        count--;//队列元素减1
        if (itrs != null)
            itrs.removedAt(removeIndex);//更新迭代器数据
    }
    notFull.signal();//唤醒添加线程
}
```

### 缺点

a）队列长度固定且必须在初始化时指定，所以使用之前一定要慎重考虑好容量；

b）如果消费速度跟不上入队速度，则会导致提供者线程一直阻塞，且越阻塞越多，非常危险；

c）只使用了一个锁来控制入队出队，效率较低。