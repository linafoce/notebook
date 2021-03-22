### 你用的 Mysql 是哪个引擎，各引擎之间有什么区别？(2018-4-16-lxy)

主要 MyISAM 与 InnoDB 两个引擎，其主要区别如下： 

InnoDB 支持事务，MyISAM 不支持，这一点是非常之重要。事务是一种高级的处理方式，如在一 

些列增删改中只要哪个出错还可以回滚还原，而 MyISAM 就不可以了； 

MyISAM 适合查询以及插入为主的应用，InnoDB 适合频繁修改以及涉及到安全性较高的应用； 

InnoDB 支持外键，MyISAM 不支持； 

MyISAM 是默认引擎，InnoDB 需要指定； 

InnoDB 不支持 FULLTEXT 类型的索引；

InnoDB 中不保存表的行数，如 select count() from table 时，InnoDB；需要扫描一遍整个表来 

计算有多少行，但是 MyISAM 只要简单的读出保存好的行数即可。注意的是，当 count()语句包含 

where 条件时 MyISAM 也需要扫描整个表； 

对于自增长的字段，InnoDB 中必须包含只有该字段的索引，但是在 MyISAM 表中可以和其他字 

段一起建立联合索引；清空整个表时，InnoDB 是一行一行的删除，效率非常慢。MyISAM 则会重建 

表；

InnoDB 支持行锁（某些情况下还是锁整表，如 update table set a=1 where user like '%lee%'

### 事务的特性？

1、原子性(Atomicity)：事务中的全部操作在数据库中是不可分割的，要么全部完成，要么均不执行。

2、一致性(Consistency)：几个并行执行的事务，其执行结果必须与按某一顺序串行执行的结果相一致。

3、隔离性(Isolation)：事务的执行不受其他事务的干扰，事务执行的中间结果对其他事务必须是透明的。

4、持久性(Durability)：对于任意已提交事务，系统必须保证该事务对数据库的改变不被丢失，即使数据库出现故障

### JDBC

1. 加载 JDBC 驱动程序： 在连接数据库之前，首先要加载想要连接的数据库的驱动到 JVM（Java 虚拟机），这通过 java.lang.Class 类的静态方法 forName(String className)实现。 例如：

```java
try{
//加载 MySql 的驱动类
Class.forName("com.mysql.jdbc.Driver") ;
}catch(ClassNotFoundException e){
System.out.println("找不到驱动程序类 ，加载驱动失败！");
e.printStackTrace() ;
}
```

成功加载后，会将 Driver 类的实例注册到 DriverManager 类中。 

2. 提供 JDBC 连接的 URL  连接 URL 定义了连接数据库时的协议、子协议、数据源标识。 

   1. 书写形式：协议：子协议：数据源标识 协议：在 JDBC 中总是以 jdbc 开始 子协议：是桥连接的驱动程序或是数据库管理系统名称。 数据源标识：标记找到数据库来源的地址与连接端口。 例如： jdbc:mysql://localhost:3306/test? useUnicode=true&characterEncoding=gbk;useUnicode=true;（MySql 的连接 URL） 表示使用 Unicode 字符集。如果 characterEncoding 设置为 gb2312 或 GBK，本参数必须设置 为 true 。characterEncoding=gbk：字符编码方式。 

3. 创建数据库的连接 

   1.  要连接数据库，需要向 java.sql.DriverManager 请求并获得 Connection 对象，该对象就代 表一个数据库的连接。 
   2. 使用 DriverManager 的 getConnectin(String url , String username , String password )方法传 入指定的欲连接的数据库的路径、数据库的用户名和 密码来获得。

   ```java
   例如： //连接 MySql 数据库，用户名和密码都是 root
   String url = "jdbc:mysql://localhost:3306/test" ;
   String username = "root" ;
   String password = "root" ;
   try{
   Connection con = DriverManager.getConnection(url , username , password) ;
   }catch(SQLException se){
   System.out.println("数据库连接失败！");
   se.printStackTrace() ;
   }
   ```

4. 创建一个 Statement

   要执行 SQL 语句，必须获得 java.sql.Statement 实例，Statement 实例分为以下 3 种类型：

    1) 执行静态 SQL 语句。通常通过 Statement 实例实现。

    2) 执行动态 SQL 语句。通常通过 PreparedStatement 实例实现。

    3) 执行数据库存储过程。通常通过 CallableStatement 实例实现。 具体的实现方式：

   ```java
   Statement stmt = con.createStatement() ;
   PreparedStatement pstmt = con.prepareStatement(sql) ;
   CallableStatement cstmt = con.prepareCall("{CALL 
   ```

5. 执行 SQL 语句

Statement 接口提供了三种执行 SQL 语句的方法：executeQuery、executeUpdate 和 execute。 

1) ResultSet executeQuery(String sqlString)：执行查询数据库的 SQL 语句 ，返回一个结果集 （ResultSet）对象。 2) int executeUpdate(String sqlString)：用于执行 INSERT、UPDATE 或 DELETE 语句以及 SQL DDL 语句，如：CREATE TABLE 和 DROP TABLE 等。

 3) execute(sqlString):用于执行返回多个结果集、多个更新计数或二者组合的 语句。 具体实现的代码：

```java
ResultSet rs = stmt.executeQuery(“SELECT * FROM …”) ;
int rows = stmt.executeUpdate(“INSERT INTO …”) ;
boolean flag = stmt.execute(String sql) ;
```

6. 处理结果 两种情况：

   1) 执行更新返回的是本次操作影响到的记录数。

    2) 执行查询返回的结果是一个 ResultSet 对象。

     ResultSet 包含符合 SQL 语句中条件的所有行，并且它通过一套 get 方法提供了对这些行 中数据的访问。

     使用结果集（ResultSet）对象的访问方法获取数据：

```java
while(rs.next()){
String name = rs.getString(“name”) ;
String pass = rs.getString(1) ; // 此方法比较高效
}
```

7. 关闭 JDBC 对象 操作完成以后要把所有使用的 JDBC 对象全都关闭，以释放 JDBC 资源，关闭顺序和声明顺序 相反：

   1) 关闭记录集

   2) 关闭声明 

   3) 关闭连接对象

```java
if(rs != null){ // 关闭记录集
try{
rs.close() ;
}catch(SQLException e){
e.printStackTrace() ;
}
}
if(stmt != null){ // 关闭声明
try{
stmt.close() ;
}catch(SQLException e){
e.printStackTrace() ;
}
}
if(conn != null){ // 关闭连接对象
try{
conn.close() ;
}catch(SQLException e){
e.printStackTrace() ;
}
}
```

## 分库分表

场景：对于大型的互联网应用来说，数据库单表的记录行数可能达到千万级甚至是亿级，并且数据库面临着极高的并发访问。采用Master-Slave复制模式的MySQL架构，

只能够对数据库的读进行扩展，而对数据库的写入操作还是集中在Master上，并且单个Master挂载的Slave也不可能无限制多，Slave的数量受到Master能力和负载的限制。

因此，需要对数据库的吞吐能力进行进一步的扩展，以满足高并发访问与海量数据存储的需要！


      对于访问极为频繁且数据量巨大的单表来说，我们首先要做的就是减少单表的记录条数，以便减少数据查询所需要的时间，提高数据库的吞吐，这就是所谓的分表！


      在分表之前，首先需要选择适当的分表策略，使得数据能够较为均衡地分不到多张表中，并且不影响正常的查询！


      对于互联网企业来说，大部分数据都是与用户关联的，因此，用户id是最常用的分表字段。因为大部分查询都需要带上用户id，这样既不影响查询，又能够使数据较为均衡地

分布到各个表中(当然，有的场景也可能会出现冷热数据分布不均衡的情况)，如下图：

![image-20210228212052951](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210228212052951.png)

假设有一张表记录用户购买信息的订单表order，由于order表记录条数太多，将被拆分成256张表。

拆分的记录根据user_id%256取得对应的表进行存储，前台应用则根据对应的user_id%256，找到对应订单存储的表进行访问。

这样一来，user_id便成为一个必需的查询条件，否则将会由于无法定位数据存储的表而无法对数据进行访问。


注：拆分后表的数量一般为2的n次方，就是上面拆分成256张表的由来！

假设order表结构如下：

create table order_(
 order_id bigint(20) primary key auto_increment,
 user_id bigint(20),
 user_nick varchar(50),
 auction_id bigint(20),
 auction_title bigint(20),
 price bigint(20),
 auction_cat varchar(200),
 seller_id bigint(20),
 seller_nick varchar(50)
)

那么分表以后，假设user_id = 257,并且auction_id = 100,需要根据auction_id来查询对应的订单信息，则对应的SQL语句如下：
select * from order_1 where user_id=257 and auction_id = 100;

其中，order_1是根据257%256计算得出，表示分表之后的第一张order表。


二. 分库

   场景：分表能够解决单表数据量过大带来的查询效率下降的问题，但是，却无法给数据库的并发处理能力带来质的提升。面对高并发的读写访问，当数据库master

服务器无法承载写操作压力时，不管如何扩展slave服务器，此时都没有意义了。

因此，我们必须换一种思路，对数据库进行拆分，从而提高数据库写入能力，这就是所谓的分库!


    与分表策略相似，分库可以采用通过一个关键字取模的方式，来对数据访问进行路由，如下图所示：



    还是之前的订单表，假设user_id 字段的值为258，将原有的单库分为256个库，那么应用程序对数据库的访问请求将被路由到第二个库(258%256 = 2)。


三. 分库分表

    场景：有时数据库可能既面临着高并发访问的压力，又需要面对海量数据的存储问题，这时需要对数据库既采用分表策略，又采用分库策略，以便同时扩展系统的

并发处理能力，以及提升单表的查询性能，这就是所谓的分库分表。


    分库分表的策略比前面的仅分库或者仅分表的策略要更为复杂，一种分库分表的路由策略如下：


    1. 中间变量 = user_id % (分库数量 * 每个库的表数量)
    
    2. 库 = 取整数 (中间变量 / 每个库的表数量)
    
    3. 表 = 中间变量 % 每个库的表数量



同样采用user_id作为路由字段，首先使用user_id 对库数量*每个库表的数量取模，得到一个中间变量；然后使用中间变量除以每个库表的数量，取整，便得到

对应的库；而中间变量对每个库表的数量取模，即得到对应的表。


分库分表策略详细过程如下：


假设将原来的单库单表order拆分成256个库，每个库包含1024个表，那么按照前面所提到的路由策略，对于user_id=262145 的访问，路由的计算过程如下：

1.  中间变量 = 262145 % (256 * 1024) = 1
2.  库 = 取整 (1/1024) = 0
3.  表 = 1 % 1024 = 1



## 死锁

**1.**  **mysql****都有什么锁**

 

MySQL有三种锁的级别：页级、表级、行级。

**表级锁**：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。

**行级锁**：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。

**页面锁**：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

 

算法：

next KeyLocks锁，同时锁住记录(数据)，并且锁住记录前面的Gap   

Gap锁，不锁记录，仅仅记录前面的Gap

Recordlock锁（锁数据，不锁Gap）

所以其实 Next-KeyLocks=Gap锁+ Recordlock锁



**2.**  **什么情况下会造成死锁** 

所谓死锁<DeadLock>: 是指两个或两个以上的进程在执行过程中,
因争夺资源而造成的一种互相等待的现象,若无外力作用,它们都将无法推进下去.
此时称系统处于死锁状态或系统产生了死锁,这些永远在互相等竺的进程称为死锁进程.
表级锁不会产生死锁.所以解决死锁主要还是针对于最常用的InnoDB.

 

死锁的关键在于**：两个(****或以上)****的Session****加锁的顺序****不一致。**

那么对应的解决死锁问题的关键就是：让不同的session加锁有次序

 

**3.**  **一些常见的死锁案例**

 

**案例一：**

需求：将投资的钱拆成几份随机分配给借款人。

起初业务程序思路是这样的：

投资人投资后，将金额随机分为几份，然后随机从借款人表里面选几个，然后通过一条条select for update 去更新借款人表里面的余额等。

 

抽象出来就是一个session通过for循环会有几条如下的语句：

Select * from xxx where id='随机id' for update

 

基本来说，程序开启后不一会就死锁。

这可以是说最经典的死锁情形了。

 

例如两个用户同时投资，A用户金额随机分为2份，分给借款人1，2

B用户金额随机分为2份，分给借款人2，1

由于加锁的顺序不一样，死锁当然很快就出现了。

 

**对于这个问题的改进很简单，直接把所有分配到的借款人直接一次锁住就行了。**

**Select \* from xxx where id in (xx,xx,xx) for update**

**在in****里面的列表值mysql****是会自动从小到大排序，加锁也是一条条从小到大加的锁**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
例如（以下会话id为主键）：

Session1:

mysql> select * from t3 where id in (8,9) for update;

+----+--------+------+---------------------+

| id | course | name | ctime               |

+----+--------+------+---------------------+

|  8 | WA     | f    | 2016-03-02 11:36:30 |

|  9 | JX     | f    | 2016-03-01 11:36:30 |

+----+--------+------+---------------------+

2 rows in set (0.04 sec)

 

 

Session2:

select * from t3 where id in (10,8,5) for update;

锁等待中……

其实这个时候id=10这条记录没有被锁住的，但id=5的记录已经被锁住了，锁的等待在id=8的这里。

 

不信请看

Session3:

mysql> select * from t3 where id=5 for update;

锁等待中

 

Session4:

mysql> select * from t3 where id=10 for update;

+----+--------+------+---------------------+

| id | course | name | ctime               |

+----+--------+------+---------------------+

| 10 | JB     | g    | 2016-03-10 11:45:05 |

+----+--------+------+---------------------+

1 row in set (0.00 sec)

 

在其它session中id=5是加不了锁的，但是id=10是可以加上锁的。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

 

 

**案例2****：**

在开发中，经常会做这类的判断需求：根据字段值查询（有索引），如果不存在，则插入；否则更新。

 [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
以id为主键为例，目前还没有id=22的行

Session1:

select * from t3 where id=22 for update;

Empty set (0.00 sec)

 

session2:

select * from t3 where id=23  for update;

Empty set (0.00 sec)

 

Session1:

insert into t3 values(22,'ac','a',now());

锁等待中……

 

Session2:

insert into t3 values(23,'bc','b',now());

ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction

 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

当对存在的行进行锁的时候(主键)，mysql就只有行锁。

当对未存在的行进行锁的时候(即使条件为主键)，mysql是会锁住一段范围（有gap锁）



锁住的范围为：

(无穷小或小于表中锁住id的最大值，无穷大或大于表中锁住id的最小值)

 

如：如果表中目前有已有的id为（11 ， 12）

那么就锁住（12，无穷大）

如果表中目前已有的id为（11 ， 30）

那么就锁住（11，30）

 

**对于这种死锁的解决办法是：**

**insert into t3(xx,xx) on duplicate key update `xx`='XX';**

 

用mysql特有的语法来解决此问题。因为insert语句对于主键来说，插入的行不管有没有存在，都会只有行锁。

 

 

**案例3****：**

直接上情景：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
mysql> select * from t3 where id=9 for update;

+----+--------+------+---------------------+

| id | course | name | ctime               |

+----+--------+------+---------------------+

|  9 | JX     | f    | 2016-03-01 11:36:30 |

+----+--------+------+---------------------+

1 row in set (0.00 sec)

 

Session2:

mysql> select * from t3 where id<20 for update;

锁等待中

 

Session1:

mysql> insert into t3 values(7,'ae','a',now());

ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction

 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

这个跟案例一其它是差不多的情况，只是session1不按常理出牌了，

Session2在等待Session1的id=9的锁，session2又持了1到8的锁（注意9到19的范围并没有被session2锁住），最后，session1在插入新行时又得等待session2,故死锁发生了。

 

这种一般是在业务需求中基本不会出现，因为你锁住了id=9，却又想插入id=7的行，这就有点跳了，当然肯定也有解决的方法，那就是重理业务需求，避免这样的写法。

