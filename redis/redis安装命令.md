#### linux环境

yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm



##### 查看可用版本

*yum --enablerepo=remi list redis --showduplicates | sort -r*



##### 安装指定版本redis

*yum --enablerepo=remi install redis-6.0.6 -y*



#### 启动redis服务

```
# 启动redis
service redis start
# 停止redis
service redis stop
# 查看redis运行状态
service redis status
# 查看redis进程
ps` `-ef | ``grep` `redis
```

#### 修改redis配置文件

```
vi` `/etc/redis``.conf
```





#### 开放linux端口

##### 命令的方式

 /sbin/iptables -I INPUT -p tcp --dport 6379 -j ACCEPT

##### 或者修改配置文件

vi /etc/sysconfig/iptables

##### redis改成对外服务

vi /etc/redis.conf

![image-20210209192247661](/Users/xiangtingwei/Library/Application Support/typora-user-images/image-20210209192247661.png)

