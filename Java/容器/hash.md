**这个问题其实☞指的就是，hashCode的计算逻辑中，为什么是31作为乘数。**

### 1. 固定乘积31在这用到了

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

这段内容主要阐述的观点包括；

1. 31 是一个奇质数，如果选择偶数会导致乘积运算时数据溢出。
2. 另外在二进制中，2个5次方是32，那么也就是 `31 * i == (i << 5) - i`。这主要是说乘积运算可以使用位移提升性能，同时目前的JVM虚拟机也会自动支持此类的优化。

#### 3.2 Hash计算函数

```java
public static Integer hashCode(String str, Integer multiplier) {
    int hash = 0;
    for (int i = 0; i < str.length(); i++) {
        hash = multiplier * hash + str.charAt(i);
    }
    return hash;
}
```

- 这个过程比较简单，与原hash函数对比只是替换了可变参数，用于我们统计不同乘积数的计算结果。

#### 3.3 Hash碰撞概率计算

想计算碰撞很简单，也就是计算那些出现相同哈希值的数量，计算出碰撞总量即可。这里的实现方式有很多，可以使用`set`、`map`也可以使用`java8`的`stream`流统计`distinct`。

```java
private static RateInfo hashCollisionRate(Integer multiplier, List<Integer> hashCodeList) {
    int maxHash = hashCodeList.stream().max(Integer::compareTo).get();
    int minHash = hashCodeList.stream().min(Integer::compareTo).get();
    int collisionCount = (int) (hashCodeList.size() - hashCodeList.stream().distinct().count());
    double collisionRate = (collisionCount * 1.0) / hashCodeList.size();
    return new RateInfo(maxHash, minHash, multiplier, collisionCount, collisionRate);
}
```

- 这里记录了最大hash和最小hash值，以及最终返回碰撞数量的统计结果。

#### 3.4 单元测试

```java
@Before
public void before() {
    "abc".hashCode();
    // 读取文件，103976个英语单词库.txt
    words = FileUtil.readWordList("E:/itstack/git/github.com/interview/interview-01/103976个英语单词库.txt");
}

@Test
public void test_collisionRate() {
    List<RateInfo> rateInfoList = HashCode.collisionRateList(words, 2, 3, 5, 7, 17, 31, 32, 33, 39, 41, 199);
    for (RateInfo rate : rateInfoList) {
        System.out.println(String.format("乘数 = %4d, 最小Hash = %11d, 最大Hash = %10d, 碰撞数量 =%6d, 碰撞概率 = %.4f%%", rate.getMultiplier(), rate.getMinHash(), rate.getMaxHash(), rate.getCollisionCount(), rate.getCollisionRate() * 100));
    }
}
```

- 以上先设定读取英文单词表中的10个单词，之后做hash计算。
- 在hash计算中把单词表传递进去，同时还有乘积数；`2, 3, 5, 7, 17, 31, 32, 33, 39, 41, 199`，最终返回一个list结果并输出。
- 这里主要验证同一批单词，对于不同乘积数会有怎么样的hash碰撞结果。

![公众号：bugstack虫洞栈，hash碰撞图表](https://bugstack.cn/assets/images/2020/interview/interview-3-03.png)

以上就是不同的乘数下的hash碰撞结果图标展示，从这里可以看出如下信息；

1. 乘数是2时，hash的取值范围比较小，基本是堆积到一个范围内了，后面内容会看到这块的展示。
2. 乘数是3、5、7、17等，都有较大的碰撞概率
3. **乘数是31的时候，碰撞的概率已经很小了，基本稳定。**
4. 顺着往下看，你会发现199的碰撞概率更小，这就相当于一排奇数的茅坑量多，自然会减少碰撞。**但这个范围值已经远超过int的取值范围了，如果用此数作为乘数，又返回int值，就会丢失数据信息**。

### 4. Hash值散列分布

除了以上看到哈希值在不同乘数的一个碰撞概率后，关于散列表也就是hash，还有一个非常重要的点，那就是要尽可能的让数据散列分布。只有这样才能减少hash碰撞次数，也就是后面章节要讲到的hashMap源码。

那么怎么看散列分布呢？如果我们能把10万个hash值铺到图表上，形成的一张图，就可以看出整个散列分布。但是这样的图会比较大，当我们缩小看后，就成一个了大黑点。所以这里我们采取分段统计，把2 ^ 32方分64个格子进行存放，每个格子都会有对应的数量的hash值，最终把这些数据展示在图表上。

#### 4.1 哈希值分段存放

```java
public static Map<Integer, Integer> hashArea(List<Integer> hashCodeList) {
    Map<Integer, Integer> statistics = new LinkedHashMap<>();
    int start = 0;
    for (long i = 0x80000000; i <= 0x7fffffff; i += 67108864) {
        long min = i;
        long max = min + 67108864;
        // 筛选出每个格子里的哈希值数量，java8流统计；https://bugstack.cn/itstack-demo-any/2019/12/10/%E6%9C%89%E7%82%B9%E5%B9%B2%E8%B4%A7-Jdk1.8%E6%96%B0%E7%89%B9%E6%80%A7%E5%AE%9E%E6%88%98%E7%AF%87(41%E4%B8%AA%E6%A1%88%E4%BE%8B).html
        int num = (int) hashCodeList.parallelStream().filter(x -> x >= min && x < max).count();
        statistics.put(start++, num);
    }
    return statistics;
```

- 这个过程主要统计`int`取值范围内，每个哈希值存放到不同格子里的数量。
- 这里也是使用了java8的新特性语法，统计起来还是比较方便的。

#### 4.2 单元测试

```
@Test
public void test_hashArea() {
    System.out.println(HashCode.hashArea(words, 2).values());
    System.out.println(HashCode.hashArea(words, 7).values());
    System.out.println(HashCode.hashArea(words, 31).values());
    System.out.println(HashCode.hashArea(words, 32).values());
    System.out.println(HashCode.hashArea(words, 199).values());
}
```

- 这里列出我们要统计的乘数值，每一个乘数下都会有对应的哈希值数量汇总，也就是64个格子里的数量。
- 最终把这些统计值放入到excel中进行图表化展示。

**统计图表**

![公众号：bugstack虫洞栈，hash散列表](https://bugstack.cn/assets/images/2020/interview/interview-3-04.png)

- 以上是一个堆积百分比统计图，可以看到下方是不同乘数下的，每个格子里的数据统计。
- 除了199不能用以外，31的散列结果相对来说比较均匀。

##### 4.2.1 乘数2散列

![img](https://bugstack.cn/assets/images/2020/interview/interview-3-05.png)

- 乘数是2的时候，散列的结果基本都堆积在中间，没有很好的散列。

##### 4.2.2 乘数31散列

![img](https://bugstack.cn/assets/images/2020/interview/interview-3-06.png)

- 乘数是31的时候，散列的效果就非常明显了，基本在每个范围都有数据存放。

##### 4.2.3 乘数199散列

![img](https://bugstack.cn/assets/images/2020/interview/interview-3-07.png)

- 乘数是199是不能用的散列结果，但是它的数据是更加分散的，从图上能看到有两个小山包。但因为数据区间问题会有数据丢失问题，所以不能选择。

**文中引用**

- http://www.tianxiaobo.com/2018/01/18/String-hashCode-%E6%96%B9%E6%B3%95%E4%B8%BA%E4%BB%80%E4%B9%88%E9%80%89%E6%8B%A9%E6%95%B0%E5%AD%9731%E4%BD%9C%E4%B8%BA%E4%B9%98%E5%AD%90/
- https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier

## 五、总结

- 以上主要介绍了hashCode选择31作为乘数的主要原因和实验数据验证，算是一个散列的数据结构的案例讲解，在后续的类似技术中，就可以解释其他的源码设计思路了。
- 看过本文至少应该让你可以从根本上解释了hashCode的设计，关于他的所有问题也就不需要死记硬背了，学习编程内容除了最开始的模仿到深入以后就需要不断的研究数学逻辑和数据结构。
- 文中参考了优秀的hashCode资料和stackoverflow，并亲自做实验验证结果，大家也可以下载本文中资源内容；英文字典、源码、excel图表等内容。