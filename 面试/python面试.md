## GIL python

![image-20210225180826152](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210225180826152.png)

## @staticmethod和@classmethod

Python其实有3个方法,即静态方法(staticmethod),类方法(classmethod)和实例方法,如下:

```python
def foo(x):
    print "executing foo(%s)"%(x)

class A(object):
    def foo(self,x):
        print "executing foo(%s,%s)"%(self,x)

    @classmethod
    def class_foo(cls,x):
        print "executing class_foo(%s,%s)"%(cls,x)

    @staticmethod
    def static_foo(x):
        print "executing static_foo(%s)"%x

a=A()
```

## Lambda

简单来说，编程中提到的 lambda 表达式，通常是在**需要一个函数，但是又不想费神去命名一个函数**的场合下使用，也就是指**匿名函数**。这一用法跟所谓 λ 演算（题目说明里的维基链接）的关系，有点像原子弹和质能方程的关系，差别其实还是挺大的。

不谈形式化的 λ 演算，只说有实际用途的匿名函数。先举一个普通的 Python 例子：将一个 list 里的每个元素都平方：

```python
map( lambda x: x*x, [y for y in range(10)] )
```

这个写法要好过

```python
def sq(x):
    return x * x

map(sq, [y for y in range(10)])
```

，因为后者多定义了一个（污染环境的）函数，尤其如果这个函数只会使用一次的话。而且第一种写法实际上更易读，因为那个映射到列表上的函数具体是要做什么，非常一目了然。如果你仔细观察自己的代码，会发现这种场景其实很常见：你在某处就真的只需要一个能做一件事情的函数而已，连它叫什么名字都无关紧要。Lambda 表达式就可以用来做这件事。

## Python函数式编程

这个需要适当的了解一下吧,毕竟函数式编程在Python中也做了引用.

推荐: [酷壳](http://coolshell.cn/articles/10822.html)

python中函数式编程支持:

filter 函数的功能相当于过滤器。调用一个布尔函数`bool_func`来迭代遍历每个seq中的元素；返回一个使`bool_seq`返回值为true的元素的序列。

```python 
>>>a = [1,2,3,4,5,6,7]
>>>b = filter(lambda x: x > 5, a)
>>>print b
>>>[6,7]
```

map函数是对一个序列的每个项依次执行函数，下面是对一个序列每个项都乘以2：

```
>>> a = map(lambda x:x*2,[1,2,3])
>>> list(a)
[2, 4, 6]
```

reduce函数是对一个序列的每个项迭代调用函数，下面是求3的阶乘：

```
>>> reduce(lambda x,y:x*y,range(1,4))
6
```

## Python里的拷贝

引用和copy(),deepcopy()的区别

```python
import copy
a = [1, 2, 3, 4, ['a', 'b']]  #原始对象

b = a  #赋值，传对象的引用
c = copy.copy(a)  #对象拷贝，浅拷贝
d = copy.deepcopy(a)  #对象拷贝，深拷贝

a.append(5)  #修改对象a
a[4].append('c')  #修改对象a中的['a', 'b']数组对象

print 'a = ', a
print 'b = ', b
print 'c = ', c
print 'd = ', d

输出结果：
a =  [1, 2, 3, 4, ['a', 'b', 'c'], 5]
b =  [1, 2, 3, 4, ['a', 'b', 'c'], 5]
c =  [1, 2, 3, 4, ['a', 'b', 'c']]
d =  [1, 2, 3, 4, ['a', 'b']]
```

## Python的is

is是对比地址,==是对比值

## read,readline和readlines

- read 读取整个文件
- readline 读取下一行,使用生成器方法
- readlines 读取整个文件到一个迭代器以供我们遍历

## range and xrange

都在循环时使用，xrange内存性能更好。 for i in range(0, 20): for i in xrange(0, 20): What is the difference between range and xrange functions in Python 2.X? range creates a list, so if you do range(1, 10000000) it creates a list in memory with 9999999 elements. xrange is a sequence object that evaluates lazily.



### Python 生成器