# Zookeeper的客户端

Demo源码地址：https://gitee.com/archguide/zookeeper-demo

git clone地址：https://gitee.com/archguide/zookeeper-demo.git

## 原生客户端增删查改

Zookeeper自带了两个客户端：



1. 一个是命令行客户端，就是zkCli.sh/zkCli.cmd
2. 一个是Java客户端，就是Zookeeper类，也就是我说的原生客户端



### 连接服务端

```
/**
 * connectString 连接地址可以写多个，比如"127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183",当客户端与服务端的Socket连接断掉后就会重试去连其他的服务器地址
 * sessionTimeout 客户端设置的话会超时时间
 * watcher 监听器
*/

ZooKeeper zooKeeper = new ZooKeeper("192.168.40.52:2181", 60 * 1000, new Watcher() {
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }
});
```



先忽略监听器，后面详细讲



### 创建节点

```
// 创建一个节点，并设置内容，设置ACL(该节点的权限设置)，
// 节点类型（7种：持久节点、临时节点、持久顺序节点、临时顺序节点、容器节点、TTL节点、TTL顺序节点）
String result = zooKeeper.create("/luban123", "123".getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
System.out.println(result);
```



**enum** CreateMode：

- PERSISTENT：持久节点
- PERSISTENT_SEQUENTIAL：持久顺序节点
- EPHEMERAL：临时节点
- EPHEMERAL_SEQUENTIAL：临时顺序节点
- CONTAINER：容器节点
- PERSISTENT_WITH_TTL：TTL持久节点
- PERSISTENT_SEQUENTIAL_WITH_TTL：TT持久顺序节点





### 查询节点

```
Stat stat = new Stat();
byte[] data = zooKeeper.getData("/luban123", false, stat);

System.out.println(new String(data));
```



一个节点，除开有节点名字，节点内容之外，还有其他的信息，这些信息成为Stat，表示节点的状态信息，Stat包括：



![img](https://cdn.nlark.com/yuque/0/2021/jpeg/365147/1609765274771-d106de3d-f67e-44dc-8efd-453e5a405bbd.jpeg)



在查询的时候，要传入一个Stat对象用来承载待查询节点的状态信息。



### 节点是否存在

```
Stat stat = zooKeeper.exists("/luban1234", false);
System.out.println(stat);  // 如果节点不存在，则stat为null
```



### 修改节点

```
// 修改节点的内容，这里有乐观锁,version表示本次修改, -1表示不检查版本强制更新
// stat表示修改数据成功之后节点的状态
Stat stat = zooKeeper.setData("/luban", "xxx".getBytes(), -1);
```



### 删除节点

```
zooKeeper.delete("/luban123", -1);
```



### 创建子节点

```
String result = zooKeeper.create("/luban/luban123", "123".getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
System.out.println(result);
```





### 获取子节点

```
Stat stat = new Stat();
List<String> children = zooKeeper.getChildren("/luban", false, stat);
System.out.println(children);
```



## 原生客户端Watch机制



### 一次性的Watcher



给/luban节点绑定了一个监听，用来监听节点的数据变化和当前节点的删除事件

```
Stat stat = new Stat();
zooKeeper.getData("/luban", new Watcher() {
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }
}, stat);
```



给/luban节点绑定了一个监听，用来监听节点的子节点的变化，子节点的增加和删除

```
zooKeeper.getChildren("/luban", new Watcher() {
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println("child");
    }
});
```



给/luban123节点绑定了一个监听，用来监听节点的创建事件

```
zooKeeper.exists("/luban123", new Watcher() {
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }
});
```



### 持久化的Watcher



给/luban123节点绑定了一个持久化监听，可以监听节点的创建、删除、修改、增加子节点、删除子节点事件

```
zooKeeper.addWatch("/luban123", new Watcher() {
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }
}, AddWatchMode.PERSISTENT);
```



### 递归持久化的Watcher



给/luban123节点绑定了一个持久化监听，可以监听节点的创建、删除、修改、增加子节点、删除子节点事件。

并且还会监听/luban123的所有子节点的变化，包括子节点的创建、删除、修改。

```
zooKeeper.addWatch("/luban123", new Watcher() {
    @Override
    public void process(WatchedEvent watchedEvent) {
        System.out.println(watchedEvent);
    }
}, AddWatchMode.PERSISTENT_RECURSIVE);
```



## 原生客户端异步调用

```
zooKeeper.create("/luban123/xxx_", "123".getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL, new AsyncCallback.StringCallback() {
                    @Override
                    public void processResult(int i, String s, Object o, String s1) {
                        System.out.println(i); // 命令执行结果，可以参考KeeperException.Code枚举
                        System.out.println(s); // 传入的path
                        System.out.println(o); // 外部传进来的ctx
                        System.out.println(s1);// 创建成功后的path，比如顺序节点

                    }
                }, "ctx");
```



```
zooKeeper.create("/luban123/xxx_", "123".getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL, new AsyncCallback.Create2Callback() {
                    @Override
                    public void processResult(int i, String s, Object o, String s1, Stat stat) {
                        System.out.println(i);      // 命令执行结果，可以参考KeeperException.Code枚举
                        System.out.println(s);      // 传入的path
                        System.out.println(o);      // 外部传进来的ctx
                        System.out.println(s1);     // 创建成功后的path，比如顺序节点
                        System.out.println(stat);   // 创建出来的节点的stat
                    }
                }, "ctx");
```



## Curator



### 连接服务端

```
// 重连策略
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);

// 建立客户端
CuratorFramework client = CuratorFrameworkFactory.builder()
    .connectString("192.168.40.52:2181")
    .sessionTimeoutMs(60 * 1000)  // 会话超时时间
    .connectionTimeoutMs(5000) // 连接超时时间
    .retryPolicy(retryPolicy)
    .build();
client.start();
```



### 创建节点

```
String path = client.create().forPath("/luban-curator", "123".getBytes());
System.out.println(path);
```



### 查询节点

```
byte[] bytes = client.getData().forPath("/luban-curator");
System.out.println(new String(bytes));
```



### 节点是否存在

```
Stat stat = client.checkExists().forPath("/luban-curator");
System.out.println(stat);
```



### 修改节点

```
Stat stat = client.setData().forPath("/luban-curator", "456".getBytes());
System.out.println(stat);
```



### 删除节点

```
client.delete().forPath("/luban-curator");
```



### 创建子节点

```
String path = client.create().forPath("/luban-curator/test", "xxx".getBytes());
System.out.println(path);
```



### 获取子节点

```
List<String> list = client.getChildren().forPath("/luban-curator");
System.out.println(list);
```



### 创建容器节点

```
client.createContainers("/luban-curator-containers");
```



### 创建TTL节点

```
String path = client.create().withTtl(3000).withMode(CreateMode.PERSISTENT_WITH_TTL).forPath("/luban-ttl");
System.out.println(path);
```

前提是zkServer启动时得做一点而外的配置，在zkServer.sh中修改：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/365147/1609665603974-2a5971da-be0d-4f84-9b83-4c5ae35a9483.png)



## Curator中的Watch机制



### NodeCache

```
NodeCache nodeCache = new NodeCache(client, "/luban");
nodeCache.start(true);
System.out.println(nodeCache.getCurrentData());
```



NodeCache监听自身节点的数据变化

```
NodeCache nodeCache = new NodeCache(client, "/luban");
        nodeCache.start();
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println(123);
            }
        });
```



### PathChildrenCache

*
*

PathChildrenCache能够监听自身节点下的子节点的变化

```
PathChildrenCache pathChildrenCache = new PathChildrenCache(client, "/luban", true);
        pathChildrenCache.start();

        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, PathChildrenCacheEvent event) throws Exception {
                switch (event.getType()) {
                    case CHILD_ADDED: {
                        System.out.println("Node added: " + ZKPaths.getNodeFromPath(event.getData().getPath()));
                        break;
                    }

                    case CHILD_UPDATED: {
                        System.out.println("Node changed: " + ZKPaths.getNodeFromPath(event.getData().getPath()));
                        break;
                    }

                    case CHILD_REMOVED: {
                        System.out.println("Node removed: " + ZKPaths.getNodeFromPath(event.getData().getPath()));
                        break;
                    }
                }
            }
        });
```



### TreeCache



TreeCache即能够监听自身节点的变化，也能监听子节点的变化

```
TreeCache treeCache = new TreeCache(client, "/luban");
        treeCache.start();
        treeCache.getListenable().addListener(new TreeCacheListener() {
            @Override
            public void childEvent(CuratorFramework curatorFramework, TreeCacheEvent event) throws Exception {
                if (event.getData() != null) {
                    System.out.println("type=" + event.getType() + " path=" + event.getData().getPath());
                } else {
                    System.out.println("type=" + event.getType());
                }
            }
        });
```

###  

###  

### CuratorCache

这是新版curator-framework新增的特性，用来替换以上三个Cache的。



forCreates：监听监听节点的创建，和子节点的创建

forChanges：监听监听节点的内容的改变，和子节点内容的改变

forDeletes：监听监听节点的删除，和子节点的删除

```
CuratorCacheListener listener = CuratorCacheListener.builder()
    .forCreates(node -> System.out.println(String.format("Node created: [%s]", node)))
    .forChanges((oldNode, node) -> System.out.println(String.format("Node changed. Old: [%s] New: [%s]", oldNode, node)))
    .forDeletes(oldNode -> System.out.println(String.format("Node deleted. Old value: [%s]", oldNode)))
    .forInitialized(new Runnable() {
        @Override
        public void run() {
            System.out.println("Cache initialized");
        }
    })
    .build();


CuratorCache curatorCache = CuratorCache.build(client, "/luban");
curatorCache.listenable().addListener(CuratorCacheListener.builder().forAll(listener).build());
curatorCache.start();
        
System.in.read();
```





## Curator中的异步调用

```
String s = client.create().withMode(CreateMode.PERSISTENT_SEQUENTIAL).inBackground(new BackgroundCallback() {
    @Override
    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
        System.out.println(curatorEvent);
    }
}).forPath("/luban/h_", "123".getBytes());

System.out.println(s);

System.in.read();
```