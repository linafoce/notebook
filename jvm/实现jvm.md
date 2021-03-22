![image-20201124211942540](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201124211942540.png)

### jvm常用命令

![image-20201124212114391](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201124212114391.png)



## 寻找类路径

1.这里面会对应3个类加载器

![image-20201124230059573](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201124230059573.png)

启动类路径默认对应**jre\lib**目录，Java标准库（大部分在rt.jar里） 位于该路径。

扩展类路径默认对应**jre\lib\ext**目录，使用Java扩展机制的类位于这个路径。

我们自己实现的类，以及第三方类库则位于 用户类路径。

可以通过**-Xbootclasspath**选项修改启动类路径，不过 通常并不需要这样做，所以这里就不详细介绍了。



![image-20201124230546181](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201124230546181.png)

### Java存储模式

大端模式，是指数据的高字节保存在内存的低地址中，而数据的低字节保存在内存的高地址中，这样的存储模式有点儿类似于把数据当作字符串顺序处理：地址由小向大增加，而数据从高位往低位放；这和我们的阅读习惯一致。

小端模式，是指数据的高字节保存在内存的高地址中，而数据的低字节保存在内存的低地址中，这种存储模式将地址的高低和数据位权有效地结合起来，高地址部分权值高，低地址部分权值低。

![image-20201125204153381](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125204153381.png)

![image-20201125213045792](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125213045792.png)

## Access_flags

super主要为了兼容老编译器

![image-20201125221249363](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125221249363.png)

## fields

![image-20201125222041108](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125222041108.png)

## Methods

![image-20201125222059944](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125222059944.png)

## Attributes

![image-20201125222131310](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125222131310.png)



## 运行时数据区

![image-20201125230110354](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125230110354.png)

<img src="/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201126233923422.png" alt="image-20201126233923422" style="zoom:50%;" />

### 运行时数据区案例		

操作数栈的深度，局部变量表的长度，编译的时候就能够确定 

传递的参数是1.6f，所以开始的时候，第0位是1.6				![image-20201126235827142](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201126235827142.png)



#### 方法区

需要搞清楚方法区、永久代、元空间三个名词之间的关系

方法区是规范，永久代、元空间是具体实现。或者说，方法区是Java中的接口，永久代、元空间是Java中接口的实现类。



再说下永久代、元空间之间的区别



**永久代：jdk8之前方法区的实现，在堆中，用于存放类的元信息，及InstanceKlass类的实例** 

**元空间：jdk8及之后方法区的实现，在OS内存中，用于存放类的元信息**

![image-20201125231022825](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125231022825.png)

![image-20201125233912968](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20201125233912968.png)