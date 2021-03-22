## 1.基础

### 面向对象

#### 1. 封装

封装给对象提供了隐藏内部特性和行为的能力。对象提供一些能被其他对象访问的方法 来改变它内部的数据。在 Java 当中，有 3 种修饰符：public，private 和 protected。每一 种修饰符给其他的位于同一个包或者不同包下面对象赋予了不同的访问权限。

下面列出了使用封装的一些好处： 

通过隐藏对象的属性来保护对象内部的状态

  提高了代码的可用性和可维护性，因为对象的行为可以被单独的改变或者是扩展 

 禁止对象之间的不良交互提高模块化

#### 2. 继承

继承给对象提供了从基类获取字段和方法的能力。继承提供了代码的重用行，也可以在 不修改类的情况下给现存的类添加新特性。

#### 3. 多态

多态是编程语言给不同的底层数据类型做相同的接口展示的一种能力。一个多态类型上 的操作可以应用到其他类型的值上面。

#### 4. 抽象

抽象是把想法从具体的实例中分离出来的步骤，因此，要根据他们的功能而不是实现细 节来创建类。 Java 支持创建只暴漏接口而不包含方法实现的抽象的类。这种抽象技术 的主要目的是把类的行为和实现细节分离开。

### Final,finally, finalize的区别

#### 1. Final

抽象是把想法从具体的实例中分离出来的步骤，因此，要根据他们的功能而不是实现细 节来创建类。 Java 支持创建只暴漏接口而不包含方法实现的抽象的类。这种抽象技术 的主要目的是把类的行为和实现细节分离开。

#### 2. finally

在异常处理时提供 finally 块来执行任何清除操作。如果抛出一个异常，那么相匹配的 catch 子句就会执行，然后控制就会进入 finally 块（如果有的话）。

#### 3. Finalize

方法名。Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前 做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象 调用的。它是在 Object 类中定义的，因此所有的类都继承了它。子类覆盖 finalize() 方 法以整理系统资源或者执行其他清理工作。finalize() 方法是在垃圾收集器删除对象之前 对这个对象调用的。

### 3.  int 和 Integer 有什么区别

int 是基本数据类型；Integer 是其包装类，注意是一个类。

为什么要提供包装类呢？

  一是为了在各种类型间转化，通过各种方法的调用。否则 你无法直接通过变量转化。比 如，现在 int 要转为 String：

```java
int a=0;
String result=Integer.toString(a);
```

在 java 中包装类，比较多的用途是用在于各种数据类型的转化中。

```java
//通过包装类来实现转化的
int num=Integer.valueOf("12");
int num2=Integer.parseInt("12");
double num3=Double.valueOf("12.2");
double num4=Double.parseDouble("12.2");
//其他的类似。通过基本数据类型的包装来的 valueOf 和 parseXX 来实现 String 转为 XX
String a=String.valueOf("1234");//这里括号中几乎可以是任何类型
String b=String.valueOf(true);
String c=new Integer(12).toString();//
String d=new Double(2.3).toString();
```

```java
List<Integer> nums;
```

这里<>需要类。如果你用 int。它会报错的。

### 4  重载和重写

 override（重写）

1. 方法名、参数、返回值相同。
2. 子类方法不能缩小父类方法的访问权限。 
3. 子类方法不能抛出比父类方法更多的异常(但子类方法可以不抛出异常)。
4.  存在于父类和子类之间。 
5. 方法被定义为 final 不能被重写。

overload（重载）

1. 参数类型、个数、顺序至少有一个不相同。
2. 不能重载只有返回值不同的方法名。
3. 存在于父类和子类、同类中。

![image-20210227150948626](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210227150948626.png)

#### 5. 抽象类和接口有什么区别

1. 接口是公开的，里面不能有私有的方法或变量，是用于让别人使用的，而抽象类是可以 有私有方法或私有变量的； 
2. 另外，实现接口的一定要实现接口里定义的所有方法，而实现抽象类可以有选择地重写 需要用到的方法，一般的应用里，最顶级的是接口，然后是抽象类实现接口，最后才到 具体类实现； 
3. 还有，接口可以实现多重继承，而一个类只能继承一个超类，但可以通过继承多个接口 实现多重继承； 
4. 接口还有标识（里面没有任何方法，如 Remote 接口）和数据共享（里面的变量全是常 量）的作用。

#### 6. 说说反射的用途及实现

Java 反射机制主要提供了以下功能：在运行时构造一个类的对象；判断一个类所具有的成 员变量和方法；调用一个对象的方法；生成动态代理。反射最大的应用就是框架。 Java 反射的主要功能： 

1. 确定一个对象的类  取出类的 modifiers,数据成员,方法,构造器,和超类 
2. 找出某个接口里定义的常量和方法说明
3. 创建一个类实例,这个实例在运行时刻才有名字(运行时间才生成的对象) 
4. 取得和设定对象数据成员的值,如果数据成员名是运行时刻确定的也能做到
5.  在运行时刻调用动态对象的方法 
6. 创建数组,数组大小和类型在运行时刻才确定,也能更改数组成员的值

#### equals 与 == 的区别

值类型（int,char,long,boolean 等）都是用==判断相等性。对象引用的话，判断引用所指的 对象是否是同一个。equals 是 Object 的成员函数，有些类会覆盖（override）这个方法，用 于判断对象的等价性。例如 String 类，两个引用所指向的 String 都是"abc"，但可能出现他们 实际对应的对象并不是同一个（和 jvm 实现方式有关），因此用判断他们可能不相等，但用 equals 判断一定是相等的。

## 集合

### List 和 Set 区别

List,Set 都是继承自 Collection 

接口 List 特点：元素有放入顺序，元素可重复 

Set 特点：元素无放入顺序，元素不可重复，重复元素会覆盖掉 （注意：元素虽然无放入顺序，但是元素在 set 中的位置是有该元素的 HashCode 决定的， 其位置其实是固定的，加入 Set 的 Object 必须定义 equals()方法 ，另外 list 支持 for 循环， 也就是通过下标来遍历，也可以用迭代器，但是 set 只能用迭代，因为他无序，无法用下标 来取得 想要的值。）

 Set 和 List 对比：

 Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。

 List：和数组类似，List 可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起 其他元素位置改变。

### List 和 Map 区别

List 是对象集合，允许对象重复。 Map 是键值对的集合，不允许 key 重复。

### Arraylist 与 LinkedList 区别

Arraylist： 优点：ArrayList 是实现了基于动态数组的数据结构,因为地址连续，一旦数据存储好了，查 询操作效率会比较高（在内存里是连着放的）。 缺点：因为地址连续， ArrayList 要移动数据,所以插入和删除操作效率比较低。

LinkedList： 优点：LinkedList 基于链表的数据结构,地址是任意的，所以在开辟内存空间的时候不需要等 一个连续的地址，对于新增和删除操作 add 和 remove，LinedList 比较占优势。LinkedList 适 用于要头尾操作或插入指定位置的场景 缺点：因为 LinkedList 要移动指针,所以查询操作性能比较低。

适用场景分析： 当需要对数据进行对此访问的情况下选用 ArrayList，当需要对数据进行多次增加删除修改时 采用 LinkedList。

#### ArrayList 与 Vector 区别

```java
public ArrayList(int initialCapacity)//构造一个具有指定初始容量的空列表。
public ArrayList()//构造一个初始容量为 10 的空列表。
public ArrayList(Collection<? extends E> c)//构造一个包含指定 collection 的元素的列表
```

Vector 有四个构造方法：

```java
public Vector()//使用指定的初始容量和等于零的容量增量构造一个空向量。
public Vector(int initialCapacity)//构造一个空向量，使其内部数据数组的大小，其标准容量增
量为零。
public Vector(Collection<? extends E> c)//构造一个包含指定 collection 中 的元素的向量
public Vector(int initialCapacity,int capacityIncrement)//使用指定的初始容量和容量增量构造
一个空的向量
```

ArrayList 和 Vector 都是用数组实现的，主要有这么四个区别：

 (1) Vector 是多线程安全的，线程安全就是说多线程访问同一代码，不会产生不确定的结果。 而 ArrayList 不是，这个可以从源码中看出，Vector 类中的方法很多有 synchronized 进行 修饰，这样就导致了 Vector 在效率上无法与 ArrayList 相比；

 (2) 两个都是采用的线性连续空间存储元素，但是当空间不足的时候，两个类的增加方式不 同。

 (3) Vector 可以设置增长因子，而 ArrayList 不可以。

 (4) Vector 是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。

 适用场景分析： 

1) Vector 是线程同步的，所以它也是线程安全的，而 ArrayList 是线程异步的，是不安全的。 如果不考虑到线程的安全因素，一般用 ArrayList 效率比较高。

 2) 如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数 据，用 Vector 有一定的优势。

#### HashSet 和 HashMap 区别

1) set 是线性结构，set 中的值不能重复，hashset 是 set 的 hash 实现，hashset 中值不能重 复是用 hashmap 的 key 来实现的。 

2) map 是键值对映射，可以空键空值。HashMap 是 Map 接口的 hash 实现，key 的唯一性 是通过 key 值 hash 值的唯一来确定，value 值是则是链表结构。

 3) 他们的共同点都是 hash 算法实现的唯一性，他们都不能持有基本类型，只能持有对象

TreeMap：非线程安全基于红黑树实现。

 TreeMap 没有调优选项，因为该树总处于平衡状态。 

Treemap：适用于按自然顺序或自定义顺序遍历键(key)。

####  HashMap 和 ConcurrentHashMap 的区别

1) set 是线性结构，set 中的值不能重复，hashset 是 set 的 hash 实现，hashset 中值不能重 复是用 hashmap 的 key 来实现的。 

2) map 是键值对映射，可以空键空值。HashMap 是 Map 接口的 hash 实现，key 的唯一性 是通过 key 值 hash 值的唯一来确定，value 值是则是链表结构。

 3) 他们的共同点都是 hash 算法实现的唯一性，他们都不能持有基本类型，只能持有对象

ConcurrentHashMap 的工作原理及代码实现 

HashTable 里使用的是 synchronized 关键字，这其实是对对象加锁，锁住的都是对象整体， 当 Hashtable 的大小增加到一定的时候，性能会急剧下降，因为迭代时需要被锁定很长的时 间。 ConcurrentHashMap 算是对上述问题的优化，其构造函数如下，默认传入的是 16，0.75，16。

```java
public ConcurrentHashMap(int paramInt1, float paramFloat, int paramInt2) {
//…
int i = 0;
int j = 1;
while (j < paramInt2) {
++i;
j <<= 1;
}
this.segmentShift = (32 ‐ i);
this.segmentMask = (j ‐ 1);
this.segments = Segment.newArray(j);
//…
int k = paramInt1 / j;
if (k * j < paramInt1)
++k;
int l = 1;
while (l < k)
l <<= 1;
for (int i1 = 0; i1 < this.segments.length; ++i1)
this.segments[i1] = new Segment(l, paramFloat);
}

public V put(K paramK, V paramV) {
if (paramV == null)
throw new NullPointerException();
int i = hash(paramK.hashCode()); //这里的 hash 函数和 HashMap 中的不一样
return this.segments[(i >>> this.segmentShift & this.segmentMask)].put(paramK, i, paramV, false);
}
```

ConcurrentHashMap 引入了分割(Segment)，上面代码中的最后一行其实就可以理解为把一个 大的 Map 拆分成 N 个小的 HashTable，在 put 方法中，会根据 hash(paramK.hashCode())来决 定具体存放进哪个 Segment，如果查看 Segment 的 put 操作，我们会发现内部使用的同步机 制是基于 lock 操作的，这样就可以对 Map 的一部分（Segment）进行上锁，这样影响的只 是将要放入同一个 Segment 的元素的 put 操作，保证同步的时候，锁住的不是整个 Map （HashTable 就是这么做的），相对于 HashTable 提高了多线程环境下的性能，因此 HashTable 已经被淘汰了。



## 线程

### 创建线程的方式及实现

Java 中创建线程主要有三种方式： 

**1. 继承 Thread 类创建线程类** 

（1）定义 Thread 类的子类，并重写该类的 run 方法，该 run 方法的方法体就代表了线程要 完成的任务。因此把 run()方法称为执行体。 

（2）创建 Thread 子类的实例，即创建了线程对象。 

（3）调用线程对象的 start()方法来启动该线程。

```java
package com.thread;
public class FirstThreadTest extends Thread{
int i = 0; 5
//重写 run 方法，run 方法的方法体就是现场执行体
public void run() {
for(;i<100;i++){
System.out.println(getName()+" "+i);
}
}
public static void main(String[] args) {
	for(int i = 0;i< 100;i++){
			System.out.println(Thread.currentThread().getName()+" : "+i);
			if(i==20)
		{
	new FirstThreadTest().start();
	new FirstThreadTest().start();
		}
	}
	}
}
```

2. **通过 Runnable 接口创建线程类**

（1）定义 runnable 接口的实现类，并重写该接口的 run()方法，该 run()方法的方法体同样 是该线程的线程执行体。 （2）创建 Runnable 实现类的实例，并依此实例作为 Thread 的 target 来创建 Thread 对象， 该 Thread 对象才是真正的线程对象。 

（3）调用线程对象的 start()方法来启动该线程。

```java
package com.thread;
public class RunnableThreadTest implements Runnable {
private int i;
public void run()
{
for(i = 0;i <100;i++)
{
System.out.println(Thread.currentThread().getName()+" "+i);
}
}
public static void main(String[] args)
{
for(int i = 0;i < 100;i++)
{
System.out.println(Thread.currentThread().getName()+" "+i);
if(i==20)
{
RunnableThreadTest rtt = new RunnableThreadTest();
new Thread(rtt,"新线程 1").start();
new Thread(rtt,"新线程 2").start();
}
}
}
}
```

3. 通过 Callable 和 Future 创建线程 

   （1）创建 Callable 接口的实现类，并实现 call()方法，该 call()方法将作为线程执行体，并且 有返回值。 

   （2）创建 Callable 实现类的实例，使用 FutureTask 类来包装 Callable 对象，该 FutureTask 对 象封装了该 Callable 对象的 call()方法的返回值。 

   （3）使用 FutureTask 对象作为 Thread 对象的 target 创建并启动新线程。 

   （4）调用 FutureTask 对象的 get()方法来获得子线程执行结束后的返回值

```java
package com.thread;
		import java.util.concurrent.Callable;
		import java.util.concurrent.ExecutionException;
		import java.util.concurrent.FutureTask;

public class CallableThreadTest implements Callable<Integer>{
	public static void main(String[] args)
	{
		CallableThreadTest ctt = new CallableThreadTest();
		FutureTask<Integer> ft = new FutureTask<>(ctt);
		for(int i = 0;i < 100;i++) {
			System.out.println(Thread.currentThread().getName()+" 的循 环变量 i 的值"+i);
			if(i==20) {
				new Thread(ft,"有返回值的线程").start();
			}
		}
		try
		{
			System.out.println("子线程的返回值："+ft.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}
	@Override
	public Integer call() throws Exception{
		int i = 0;
		for(;i<100;i++) {
			System.out.println(Thread.currentThread().getName()+" "+i);
		}
		return i;
	}
}
```

创建线程的三种方式的对比： 

采用实现 Runnable、Callable 接口的方式创见多线程时，优势是： 线程类只是实现了 Runnable 接口或 Callable 接口，还可以继承其他类。 在这种方式下，多个线程可以共享同一个 target 对象，所以非常适合多个相同线程来处理同 一份资源的情况，从而可以将 CPU、代码和数据分开，形成清晰的模型，较好地体现了面向 对象的思想。 劣势是： 编程稍微复杂，如果要访问当前线程，则必须使用 Thread.currentThread()方法。

使用继承 Thread 类的方式创建多线程时优势是： 

编写简单，如果需要访问当前线程，则无需使用 Thread.currentThread()方法，直接使用 this 即可获得当前线程。 

劣势是：线程类已经继承了 Thread 类，所以不能再继承其他父类。

### sleep() 、join()、yield()有什么区别

1. sleep()方法 在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度 程序精度和准确性的影响。 让其他线程有机会继续执行，但它**并不释放对象锁**。也就是如 果有 Synchronized 同步块，其他线程仍然不能访问共享数据。注意该方法要捕获异常比如有 两个线程同时执行(没有 Synchronized)，一个线程优先级为 MAX_PRIORITY，另一个为 MIN_PRIORITY，如果没有 Sleep()方法，只有高优先级的线程执行完成后，低优先级的线程 才能执行；但当高优先级的线程 sleep(5000)后，低优先级就有机会执行了。 总之，sleep()可以使低优先级的线程得到执行的机会，当然也可以让同优先级、高优先级的 线程有执行的机会。
2. yield()方法 yield()方法和 sleep()方法类似，也不会释放“锁标志”，区别在于，它没有参数，即 yield() 方法只是使当前线程重新回到可执行状态，所以执行 yield()的线程有可能在进入到可执行状 态后马上又被执行，另外 yield()方法只能使同优先级或者高优先级的线程得到执行机会，这 也和 sleep()方法不同。
3.  join()方法 Thread 的非静态方法 join()让一个线程 B“加入”到另外一个线程 A 的尾部。在 A 执行完毕 之前，B 不能工作。

```
Thread t = new MyThread();
t.start();
t.join();
```

### 说说 CountDownLatch 原理

【有笔记】

### **说说** **Semaphore** **原理** 



### **说说** **Exchanger** **原理** 



### **说说** **CountDownLatch** **与** **CyclicBarrier** **区别**

![image-20210227233622811](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210227233622811.png)



### **ThreadLocal** **原理分析** 

【有笔记】

### **讲讲线程池的实现原理** 



### 