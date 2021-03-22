Redis字符串底层

c实现的,c的自己的字符串实现方式 char[] 数组

Redis实现方式 SDS simple dynamic string

1. 提供二进制安全的数据结构

2. 提供内存预先分配的机制,避免了频繁的内存分配
3. 兼容c语言函数库

字符串:

redis:

​	sds结构体:

![image-20210221201602547](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210221201602547.png)

```
	free: 0

​	len: 6

​ Char[] buf = 'guojia' 

#执行命令，str扩容
append str 123

#成倍扩容
len: 6
addlen: 3
(len + addlen) * 2 = 18


```



k-v:

Map -> dict

数据库: 海量数据存储

​	数组: O:(1)

​	链表: O(N)

​	创建一个大叔组： arr[4]

​	hash（key) -> 自然数 %4

	1. 任意相同的输入一定相同的输出
 	2. 不通的输入hash出来的值有可能一样，hash碰撞

**redis主体结构**

![image-20210221195622870](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210221195622870.png)



### list数据结构

List是一个有序(按加入的时序排序)的数据结构，Redis采用quicklist（双端链表） 和 ziplist 作为List的底层实现。

可以通过设置每个ziplist的最大容量，quicklist的数据压缩范围，提升数据存取效率

**list-max-ziplist-size  -2**       

 //  单个ziplist节点最大能存储  8kb  ,超过则进行分裂,将数据存储在新的ziplist节点中
**list-compress-depth  1**        

//  0 代表所有节点，都不进行压缩，1， 代表从头节点往后走一个，尾节点往前走一个不用压缩，其他的全部压缩，2，3，4 ... 以此类推

#### entry里面的quicklist

![image-20210221202522584](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210221202522584.png)