# ArrayList

### 特性

实现了三个标记接口：RandomAccess, Cloneable, java.io.Serializable

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

#### 1、RandomAccess

支持随机访问（基于下标）,为了能够更好地判断集合是ArrayList还是LinkedList，从而能够更好选择更优的遍历方式，提高性能！

#### 2、Cloneable

支持拷贝：实现Cloneable接口，重写clone方法、方法内容默认调用父类的clone方法。

##### 2.1、浅拷贝

​     **基础类型的变量拷贝之后是独立的，不会随着源变量变动而变**

​     **String类型拷贝之后也是独立的**

​     **引用类型不是独立的**， 引用类型拷贝的是引用地址，拷贝前后的变量引用同一个堆中的对象

```
public Object clone() throws CloneNotSupportedException {
    Study s = (Study) super.clone();
    return s;
}
```

##### 2.2、深拷贝

​		变量的所有引用类型变量（除了String）都需要实现Cloneable（数组可以直接调用clone方法），clone方法中，引用类型需要各

​     自调用clone，重新赋值

```java
public Object clone() throws CloneNotSupportedException {
    Study s = (Study) super.clone();
    // 引用对象都拷贝了一份，变成了独立的
    s.setScore(this.score.clone());
    return s;
}
```

   

java的传参，基本类型和引用类型传参

java在方法传递参数时，是将变量复制一份，然后传入方法体去执行。复制的是栈中的内容

所以基本类型是复制的变量名和值，值变了不影响源变量

引用类型复制的是变量名和值（引用地址），对象变了，会影响源变量（引用地址是一样的）

**String：是不可变对象，重新赋值时，会在常量表新生成字符串（如果已有，直接取他的引用地址），将新字符串的引用地址赋值给栈中的新变量，因此源变量不会受影响**

#### 3、Serializable

序列化：将对象状态转换为可保持或传输的格式的过程。与序列化相对的是反序列化，它将流转换为对象。这两个过程结合起来，可以轻松地存储和传输数据，在Java中的这个Serializable接口其实是给jvm看的，通知jvm，我不对这个类做序列化了，你(jvm)帮我序列化就好了。如果我们没有自己声明一个serialVersionUID变量,接口会默认生成一个serialVersionUID，默认的serialVersinUID对于class的细节非常敏感，反序列化时可能会导致InvalidClassException这个异常（每次序列化都会重新计算该值）

## 四、源码分析

### 1. 初始化

```java
List<String> list = new ArrayList<String>(10);

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

 /**
  * Constructs an empty list with the specified initial capacity.
  *
  * @param  initialCapacity  the initial capacity of the list
  * @throws IllegalArgumentException if the specified initial capacity
  *         is negative
  */
 public ArrayList(int initialCapacity) {
     if (initialCapacity > 0) {
         this.elementData = new Object[initialCapacity];
     } else if (initialCapacity == 0) {
         this.elementData = EMPTY_ELEMENTDATA;
     } else {
         throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
     }
 }
```

- 通常情况空构造函数初始化ArrayList更常用，这种方式数组的长度会在第一次插入数据时候进行设置。
- 当我们已经知道要填充多少个元素到ArrayList中，比如500个、1000个，那么为了提供性能，减少ArrayList中的拷贝操作，这个时候会直接初始化一个预先设定好的长度。
- 另外，`EMPTY_ELEMENTDATA` 是一个定义好的空对象；`private static final Object[] EMPTY_ELEMENTDATA = {}`

#### 1.1 方式01；普通方式

```
ArrayList<String> list = new ArrayList<String>();
list.add("aaa");
list.add("bbb");
list.add("ccc");
```

- 这个方式很简单也是我们最常用的方式。

#### 1.2 方式02；内部类方式

```
ArrayList<String> list = new ArrayList<String>() \\{
    add("aaa");
    add("bbb");
    add("ccc");
\\};
```

- 这种方式也是比较常用的，而且省去了多余的代码量。

#### 1.3 方式03；Arrays.asList

```
 ArrayList<String> list = new ArrayList<String>(Arrays.asList("aaa", "bbb", "ccc"));
```

以上是通过`Arrays.asList`传递给`ArrayList`构造函数的方式进行初始化，这里有几个知识点；

##### 1.3.1 ArrayList构造函数

```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

- 通过构造函数可以看到，只要实现`Collection`类的都可以作为入参。
- 在通过转为数组以及拷贝`Arrays.copyOf`到`Object[]`集合中在赋值给属性`elementData`。

**注意：c.toArray might (incorrectly) not return Object[] (`see 6260652`)**

see 6260652 是JDK bug库的编号，有点像商品sku，bug地址:https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6260652

那这是个什么bug呢，我们来测试下面这段代码；

```java
@Test
public void t(){
    List<Integer> list1 = Arrays.asList(1, 2, 3);
    System.out.println("通过数组转换：" + (list1.toArray().getClass() == Object[].class));
    
    ArrayList<Integer> list2 = new ArrayList<Integer>(Arrays.asList(1, 2, 3));
    System.out.println("通过集合转换：" + (list2.toArray().getClass() == Object[].class));
}
```

测试结果：

```
通过数组转换：false
通过集合转换：true

Process finished with exit code 0
```

- `public Object[] toArray()` 返回的类型不一定就是 `Object[]`，其类型取决于其返回的实际类型，毕竟 Object 是父类，它可以是其他任意类型。
- 子类实现和父类同名的方法，仅仅返回值不一致时，默认调用的是子类的实现方法。

造成这个结果的原因，如下；

1. Arrays.asList 使用的是：`Arrays.copyOf(this.a, size,(Class<? extends T[]>) a.getClass());`
2. ArrayList 构造函数使用的是：`Arrays.copyOf(elementData, size, Object[].class);`

##### 1.3.2 Arrays.asList

**你知道吗？**

- Arrays.asList 构建的集合，不能赋值给 ArrayList
- Arrays.asList 构建的集合，不能再添加元素
- Arrays.asList 构建的集合，不能再删除元素

那这到底为什么呢，因为Arrays.asList构建出来的List与new ArrayList得到的List，压根就不是一个List！类关系图如下；

![小傅哥 bugstack.cn & List类关系图](https://bugstack.cn/assets/images/2020/interview/interview-8-02.png)

从以上的类图关系可以看到；

1. 这两个List压根不同一个东西，而且Arrasys下的List是一个私有类，只能通过asList使用，不能单独创建。
2. 另外还有这个ArrayList不能添加和删除，主要是因为它的实现方式，可以参考Arrays类中，这部分源码；`private static class ArrayList<E> extends AbstractList<E> implements RandomAccess, java.io.Serializable`

*此外，Arrays是一个工具包，里面还有一些非常好用的方法，例如；二分查找`Arrays.binarySearch`、排序`Arrays.sort`等*

#### 1.4 方式04；Collections.ncopies

`Collections.nCopies` 是集合方法中用于生成多少份某个指定元素的方法，接下来就用它来初始化ArrayList，如下；

```
ArrayList<Integer> list = new ArrayList<Integer>(Collections.nCopies(10, 0));
```

- 这会初始化一个由10个0组成的集合。

### 2. 插入

ArrayList对元素的插入，其实就是对数组的操作，只不过需要特定时候扩容。

#### 2.1 普通插入

```
List<String> list = new ArrayList<String>();
list.add("aaa");
list.add("bbb");
list.add("ccc");
```

当我们依次插入添加元素时，ArrayList.add方法只是把元素记录到数组的各个位置上了，源码如下；

```
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

- 这是插入元素时候的源码，`size++`自增，把对应元素添加进去。

#### 2.2 插入时扩容

在前面`初始化`部分讲到，ArrayList默认初始化时会申请10个长度的空间，如果超过这个长度则需要进行扩容，那么它是怎么扩容的呢？

从根本上分析来说，数组是定长的，如果超过原来定长长度，扩容则需要申请新的数组长度，并把原数组元素拷贝到新数组中，如下图；

![小傅哥 bugstack.cn & 数组扩容](https://bugstack.cn/assets/images/2020/interview/interview-8-03.png)

图中介绍了当List结合可用空间长度不足时则需要扩容，这主要包括如下步骤；

1. 判断长度充足；`ensureCapacityInternal(size + 1);`

2. 当判断长度不足时，则通过扩大函数，进行扩容；`grow(int minCapacity)`

3. 扩容的长度计算；

   ```
   int newCapacity = oldCapacity + (oldCapacity >> 1);
   ```

   ，旧容量 + 旧容量右移1位，这相当于扩容了原来容量的

   ```
   (int)3/2
   ```

   。

   1. 10，扩容时：1010 + 1010 » 1 = 1010 + 0101 = 10 + 5 = 15
   2. 7，扩容时：0111 + 0111 » 1 = 0111 + 0011 = 7 + 3 = 10

4. 当扩容完以后，就需要进行把数组中的数据拷贝到新数组中，这个过程会用到` Arrays.copyOf(elementData, newCapacity);`，但他的底层用到的是；`System.arraycopy`

**System.arraycopy；**

```
@Test
public void test_arraycopy() {
    int[] oldArr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int[] newArr = new int[oldArr.length + (oldArr.length >> 1)];
    System.arraycopy(oldArr, 0, newArr, 0, oldArr.length);
    
    newArr[11] = 11;
    newArr[12] = 12;
    newArr[13] = 13;
    newArr[14] = 14;
    
    System.out.println("数组元素：" + JSON.toJSONString(newArr));
    System.out.println("数组长度：" + newArr.length);
    
    /**
     * 测试结果
     * 
     * 数组元素：[1,2,3,4,5,6,7,8,9,10,0,11,12,13,14]
     * 数组长度：15
     */
}
```

- 拷贝数组的过程并不复杂，主要是对`System.arraycopy`的操作。
- 上面就是把数组`oldArr`拷贝到`newArr`，同时新数组的长度，采用和ArrayList一样的计算逻辑；`oldArr.length + (oldArr.length >> 1)`

#### 2.3 指定位置插入

```
list.add(2, "1");
```

到这，终于可以说说`谢飞机`的面试题，这段代码输出结果是什么，如下；

```
Exception in thread "main" java.lang.IndexOutOfBoundsException: Index: 2, Size: 0
	at java.util.ArrayList.rangeCheckForAdd(ArrayList.java:665)
	at java.util.ArrayList.add(ArrayList.java:477)
	at org.itstack.interview.test.ApiTest.main(ApiTest.java:14)
```

**其实**，一段报错提示，为什么呢？我们翻开下源码学习下。

##### 2.3.1 容量验证

```
public void add(int index, E element) {
    rangeCheckForAdd(index);
    
    ...
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

- 指定位置插入首先要判断`rangeCheckForAdd`，size的长度。
- 通过上面的元素插入我们知道，每插入一个元素，size自增一次`size++`。
- 所以即使我们申请了10个容量长度的ArrayList，但是指定位置插入会依赖于size进行判断，所以会抛出`IndexOutOfBoundsException`异常。

##### 2.3.2 元素迁移

![小傅哥 bugstack.cn & 插入元素迁移](https://bugstack.cn/assets/images/2020/interview/interview-8-04.png)

指定位置插入的核心步骤包括；

1. 判断size，是否可以插入。
2. 判断插入后是否需要扩容；`ensureCapacityInternal(size + 1);`。
3. 数据元素迁移，把从待插入位置后的元素，顺序往后迁移。
4. 给数组的指定位置赋值，也就是把待插入元素插入进来。

**部分源码：**

```
public void add(int index, E element) {
	...
	// 判断是否需要扩容以及扩容操作
	ensureCapacityInternal(size + 1);
    // 数据拷贝迁移，把待插入位置空出来
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 数据插入操作                  
    elementData[index] = element;
    size++;
}
```

- 这部分源码的主要核心是在，`System.arraycopy`，上面我们已经演示过相应的操作方式。
- 这里只是设定了指定位置的迁移，可以把上面的案例代码复制下来做测试验证。

**实践：**

```
List<String> list = new ArrayList<String>(Collections.nCopies(9, "a"));
System.out.println("初始化：" + list);

list.add(2, "b");
System.out.println("插入后：" + list);
```

**测试结果：**

```
初始化：[a, a, a, a, a, a, a, a, a]
插入后：[a, a, 1, a, a, a, a, a, a, a]

Process finished with exit code 0
```

- 指定位置已经插入元素`1`，后面的数据向后迁移完成。

### 3. 删除

有了指定位置插入元素的经验，理解删除的过长就比较容易了，如下图；

![小傅哥 bugstack.cn & 删除元素](https://bugstack.cn/assets/images/2020/interview/interview-8-05.png)

这里我们结合着代码：

```
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

删除的过程主要包括；

1. 校验是否越界；`rangeCheck(index);`
2. 计算删除元素的移动长度`numMoved`，并通过`System.arraycopy`自己把元素复制给自己。
3. 把结尾元素清空，null。

**这里我们做个例子：**

```java
@Test
public void test_copy_remove() {
    int[] oldArr = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int index = 2;
    int numMoved = 10 - index - 1;
    System.arraycopy(oldArr, index + 1, oldArr, index, numMoved);
    System.out.println("数组元素：" + JSON.toJSONString(oldArr));
}
```

- 设定一个拥有10个元素的数组，同样按照ArrayList的规则进行移动元素。
- 注意，为了方便观察结果，这里没有把结尾元素设置为null。

**测试结果：**

```
数组元素：[1,2,4,5,6,7,8,9,10,10]

Process finished with exit code 0
```

- 可以看到指定位置 index = 2，元素已经被删掉。
- 同时数组已经移动用`元素4`占据了原来`元素3`的位置，同时结尾的10还等待删除。*这就是为什么ArrayList中有这么一句代码；elementData[–size] = null



### 迭代器

```java
private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            // 如果我没做操作之前，modcount已经变了，说明被操作过了，提前失败
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```



### 4. 扩展

如果给你一组元素；`a、b、c、d、e、f、g`，需要你放到ArrayList中，但是要求获取一个元素的时间复杂度都是O(1)，你怎么处理？

想解决这个问题，就需要知道元素添加到集合中后知道它的位置，而这个位置呢，其实可以通过哈希值与集合长度与运算，得出存放数据的下标，如下图；

![小傅哥 bugstack.cn & 下标计算](https://bugstack.cn/assets/images/2020/interview/interview-8-06.png)

- 如图就是计算出每一个元素应该存放的位置，这样就可以O(1)复杂度获取元素。

#### 4.1 代码操作(添加元素)

```
List<String> list = new ArrayList<String>(Collections.<String>nCopies(8, "0"));

list.set("a".hashCode() & 8 - 1, "a");
list.set("b".hashCode() & 8 - 1, "b");
list.set("c".hashCode() & 8 - 1, "c");
list.set("d".hashCode() & 8 - 1, "d");
list.set("e".hashCode() & 8 - 1, "e");
list.set("f".hashCode() & 8 - 1, "f");
list.set("g".hashCode() & 8 - 1, "g");
```

- 以上是初始化`ArrayList`，并存放相应的元素。存放时候计算出每个元素的下标值。

#### 4.2 代码操作(获取元素)

```
System.out.println("元素集合:" + list);
System.out.println("获取元素f [\"f\".hashCode() & 8 - 1)] Idx：" + ("f".hashCode() & (8 - 1)) + " 元素：" + list.get("f".hashCode() & 8 - 1));
System.out.println("获取元素e [\"e\".hashCode() & 8 - 1)] Idx：" + ("e".hashCode() & (8 - 1)) + " 元素：" + list.get("e".hashCode() & 8 - 1));
System.out.println("获取元素d [\"d\".hashCode() & 8 - 1)] Idx：" + ("d".hashCode() & (8 - 1)) + " 元素：" + list.get("d".hashCode() & 8 - 1));
```

#### 4.3 测试结果

```
元素集合:[0, a, b, c, d, e, f, g]

获取元素f ["f".hashCode() & 8 - 1)] Idx：6 元素：f
获取元素e ["e".hashCode() & 8 - 1)] Idx：5 元素：e
获取元素d ["d".hashCode() & 8 - 1)] Idx：4 元素：d

Process finished with exit code 0
```

- 通过测试结果可以看到，下标位置0是初始元素，元素是按照指定的下标进行插入的。
- 那么现在获取元素的时间复杂度就是O(1)，是不有点像`HashMap`中的桶结构。

## 五、总结

- 就像我们开头说的一样，数据结构是你写出代码的基础，更是写出高级代码的核心。只有了解好数据结构，才能更透彻的理解程序设计。*并不是所有的逻辑都是for循环*
- 面试题只是引导你学习的点，但不能为了面试题而忽略更重要的核心知识学习，背一两道题是不可能抗住深度问的。因为任何一个考点，都不只是一种问法，往往可以从很多方面进行提问和考查。就像你看完整篇文章，是否理解了没有说到的知识，当你固定位置插入数据时会进行数据迁移，那么在拥有大量数据的ArrayList中是不适合这么做的，非常影响性能。
- 在本章的内容编写的时候也参考到一些优秀的资料，尤其发现这份外文文档；https://beginnersbook.com/ 大家可以参考学习。