# 【9.6】LinkedList

### 特性

```java
public class LinkedList<E> extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

1、继承于 AbstractSequentialList ，本质上面与继承 AbstractList 没有什么区别，AbstractSequentialList 完善了 AbstractList 中没有实现的方法。

2、Serializable：成员变量 Node 使用 transient 修饰，通过重写read/writeObject 方法实现序列化。

3、Cloneable：重写clone（）方法，通过创建新的LinkedList 对象，遍历拷贝数据进行对象拷贝。

**4、Deque：实现了Collection 大家庭中的队列接口，说明他拥有作为双端队列的功能。**

LinkedList与ArrayList最大的区别就是LinkedList中实现了Collection中的 **Queue（Deque）接口 拥有作为双端队列的功能**

### 基本属性

链表没有长度限制，他的内存地址不需要分配固定长度进行存储，只需要记录下一个节点的存储地址即可完成整个链表的连续。

```java
//当前有多少个结点，元素个数
transient int size = 0;
//第一个结点
transient Node<E> first;
//最后一个结点
transient Node<E> last;
//Node的数据结构
private static class Node<E> {
    E item;//存储元素
    Node<E> next;//后继
    Node<E> prev;//前驱
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

LinkedList 在1.6 版本以及之前，只通过一个 header 头指针保存队列头和尾。这种操作可以说很有深度，但是从代码阅读性来说，却加深了阅读代码的难度。因此在后续的JDK 更新中，将头节点和尾节点 区分开了。节点类也更名为 Node。

为什么Node这个类是静态的？答案是：这跟内存泄露有关，Node类是在LinkedList类中的，也就是一个内部类，若不使用static修饰，那么Node就是一个普通的内部类，在java中，一个普通内部类在实例化之后，默认会持有外部类的引用，这就有可能造成内存泄露（内部类与外部类生命周期不一致时）。但使用static修饰过的内部类（称为静态内部类），就不会有这种问题



非静态内部类会自动生成一个构造器依赖于外部类：也是内部类可以访问外部类的实例变量的原因

静态内部类不会生成，访问不了外部类的实例变量，只能访问类变量

### 构造器

```java
public LinkedList() {
}
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);//操作次数只会记录一次   设置前驱后继
}
```

### 添加元素

 

```java
public boolean add(E e) {
     linkLast(e);
     return true;
 }
//目标节点创建后寻找前驱节点， 前驱节点存在就修改前驱节点的后继，指向目标节点
void linkLast(E e) {
    final Node<E> l = last;//获取这个list对象内部的Node类型成员last，即末位节点，以该节点作为新插入元素的前驱节点
    final Node<E> newNode = new Node<>(l, e, null);//创建新节点
    last = newNode;//把新节点作为该list对象的最后一个节点
    if (l == null)//处理原先的末位节点，如果这个list本来就是一个空的链表
        first = newNode;//把新节点作为首节点
    else
        l.next = newNode;//如果链表内部已经有元素，把原来的末位节点的后继指向新节点，完成链表修改
    size++;//修改当前list的size，
    modCount++;//并记录该list对象被执行修改的次数
}
public void add(int index, E element) {
    checkPositionIndex(index);//检查下标的合法性
    if (index == size)//插入位置是末位，那还是上面末位添加的逻辑
        linkLast(element);
    else
        linkBefore(element, node(index));
}
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}
Node<E> node(int index) {
    if (index < (size >> 1)) {//二分查找   index离哪端更近 就从哪端开始找
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;//找到index位置的元素
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
//指位添加方法核心逻辑  操作新节点，紧接修改原有节点的前驱属性，最后再修改前驱节点的后继属性
void linkBefore(E e, Node<E> succ) {
    final Node<E> pred = succ.prev;//原位置节点的前驱pred
    final Node<E> newNode = new Node<>(pred, e, succ);//创建新节点,设置新节点其前驱为原位置节点的前驱pred，其后继为原位置节点succ
    succ.prev = newNode;//将新节点设置到原位置节点的前驱
    if (pred == null)//前驱如果为空，空链表，则新节点设置为first
        first = newNode;
    else
        pred.next = newNode;//将新节点设置到前驱节点的后继
    size++;//修改当前list的size
    modCount++;//记录该list对象被执行修改的次数。
}
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);
    //将集合转化为数组
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Node<E> pred, succ;
    //获取插入节点的前节点（prev）和尾节点（next）
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }
    //将集合中的数据编织成链表
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }
    //将 Collection 的链表插入 LinkedList 中。
    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }
    size += numNew;
    modCount++;
    return true;
}
```

final修饰，不希望在运行时对变量做重新赋值

LinkedList 在插入数据优于ArrayList ，主要是因为他只需要修改指针的指向即可，而不需要将整个数组的数据进行转移。而LinkedList 由于没有实现 RandomAccess，或者说不支持索引搜索的原因，他在查找元素这一操作，需要消耗比较多的时间进行操作（n/2）。

### 删除元素

#### 1、AbstractSequentialList的remove

```java
public E remove(int index) {
    checkElementIndex(index);
    //node(index)找到index位置的元素
    return unlink(node(index));
}
//remove(Object o)这个删除元素的方法的形参o是数据本身，而不是LinkedList集合中的元素（节点），所以需要先通过节点遍历的方式，找到o数据对应的元素，然后再调用unlink(Node x)方法将其删除
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
E unlink(Node<E> x) {
    //x的数据域element
    final E element = x.item;
    //x的下一个结点
    final Node<E> next = x.next;
    //x的上一个结点
    final Node<E> prev = x.prev;
    //如果x的上一个结点是空结点的话，那么说明x是头结点
    if (prev == null) {
        first = next;
    } else {
        prev.next = next;//将x的前后节点相连   双向链表
        x.prev = null;//x的属性置空
    }
    //如果x的下一个结点是空结点的话，那么说明x是尾结点
    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;//将x的前后节点相连   双向链表
        x.next = null;
    }
    x.item = null;//指向null  方便GC回收
    size--;
    modCount++;
    return element;
}
```

#### 2、Deque 中的Remove

```java
//将first 节点的next 设置为新的头节点，然后将 f 清空。 removeLast 操作也类似。
private E unlinkFirst(Node<E> f) {
    final E element = f.item;
    //获取到头结点的下一个结点           
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // 方便 GC
    //头指针指向的是头结点的下一个结点
    first = next;
    //如果next为空，说明这个链表只有一个结点
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

### 双端链表（队列Queue）

java中队列的实现就是LinkedList： 我们之所以说LinkedList 为双端链表，是因为他实现了Deque 接口；我们知道，队列是先进先出的，添加元素只能从队尾添加，删除元素只能从队头删除，Queue中的方法就体现了这种特性。 支持队列的一些操作，我们来看一下有哪些方法实现：

- pop（）是栈结构的实现类的方法，返回的是栈顶元素，并且将栈顶元素删除
- poll（）是队列的数据结构，获取对头元素并且删除队头元素
- push（）是栈结构的实现类的方法，把元素压入到栈中
- peek（）获取队头元素 ，但是不删除队列的头元素
- offer（）添加队尾元素

可以看到Deque 中提供的方法主要有上述的几个方法，接下来我们来看看在LinkedList 中是如何实现这些方法的。

#### 1.1、队列的增

offer（）添加队尾元素

```
public boolean offer(E e) {
    return add(e);
}
```

具体的实现就是在尾部添加一个元素

#### 1.2、队列的删

poll（）是队列的数据结构，获取对头元素并且删除队头元素

```
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

具体的实现前面已经讲过，删除的是队列头部的元素

#### 1.3、队列的查

peek（）获取队头元素 ，但是不删除队列的头元素

```
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

#### 1.4、栈的增

push（）是栈结构的实现类的方法，把元素压入到栈中

push（） 方法的底层实现，其实就是调用了 addFirst（Object o）

```java
public void push(E e) {
    addFirst(e);
}
```

#### 1.5、栈的删

pop（）是栈结构的实现类的方法，返回的是栈顶元素，并且将栈顶元素删除

```java
public E pop() {
    return removeFirst();
}
public E removeFirst() {
    final Node f = first;
    if (f == null)
    throw new NoSuchElementException();
    return unlinkFirst(f);
}
```



普通迭代器：next

1、只能读

2、从前往后

ListIterator扩展了普通的迭代器

1、从后往前遍历

2、修改元素

3、从指定位置开始遍历

```java
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    // 双向迭代器,通过内部类自己实现迭代器
    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;//上一次 next 或者 previos 的节点
        private Node<E> next;//下一个节点
        //下一个节点的位置，从头迭代到尾，位置递增，从尾迭代到头，位置递减。
        private int nextIndex;
        //expectedModCount：期望版本号；modCount：目前最新版本号
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        // 判断还有没有下一个元素
        public boolean hasNext() {
            return nextIndex < size;//下一个节点的索引小于链表的大小，就有
        }

        // 取下一个元素
        public E next() {
            //检查期望版本号有无发生变化
            checkForComodification();
            if (!hasNext())//再次检查
                throw new NoSuchElementException();
            // next 是当前节点
            lastReturned = next;
            // next 是下一个节点了，为下次迭代做准备
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        // 如果上次节点索引位置大于 0，就还有节点可以迭代
        public boolean hasPrevious() {
            return nextIndex > 0;
        }
        // 取前一个节点
        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();
            // next 为空场景：说明是第一次迭代，取尾节点(last)
            // next 不为空场景：说明已经发生过迭代了，直接取前一个节点即可(next.prev)
            lastReturned = next = (next == null) ? last : next.prev;
            // 索引位置变化
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        public void remove() {
            checkForComodification();
            // lastReturned 为空，说明没有执行 next 或者 previos，直接报错
            // lastReturned = next() 方法执行的结果
            if (lastReturned == null)
                throw new IllegalStateException();
            Node<E> lastNext = lastReturned.next;
            //删除当前节点
            unlink(lastReturned);
            // 从尾到头递归顺序，并且是第一次迭代，并且要删除最后一个元素的情况下
            // 这种情况下，previous 方法里面设置了 lastReturned = next = last。
            // 我们必须把 next 设置成 null，这样在下次递归时，previous 方法才会让队尾最后一个节点赋值给 next
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        //since 1.8，传入一个操作方式，具体没用过这个方法(函数式编程)
        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                //accept方法传入什么类型，返回什么类型
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        //检查版本方法
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

