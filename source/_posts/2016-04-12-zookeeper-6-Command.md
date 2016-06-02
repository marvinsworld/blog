title: ZooKeeper怎么玩之六：常用命令
date: 2016-04-12 17:02:04
categories: ZooKeeper怎么玩
tags: 
---
<img src="/img/zk_cmd.jpg" class="img-topic" />
前面讲了了那么多,还没正式说如何通过命令查看zookeeper的节点和详细内容,so,这次言简意赅的说一下,zookeeper的命令也非常简单
<!--more-->
# 查看包含的内容

```java
[zk: localhost:2181(CONNECTED) 0] ls
[zk: localhost:2181(CONNECTED) 1] ls /
[config-center, config, zookeeper]
```

# 创建节点

```java
    [zk: localhost:2181(CONNECTED) 2] create /demo 测试
    Created /demo
```

# 查看节点内容

```java
[zk: localhost:2181(CONNECTED) 3] get /demo
测试
cZxid = 0x1a1
ctime = Tue Apr 12 17:54:45 CST 2016
mZxid = 0x1a1
mtime = Tue Apr 12 17:54:45 CST 2016
pZxid = 0x1a1
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

# 更新节点内容

```java
[zk: localhost:2181(CONNECTED) 0] set /demo 修改后
[zk: localhost:2181(CONNECTED) 1] get /demo 
修改后
cZxid = 0x1a1
ctime = Tue Apr 12 17:54:45 CST 2016
mZxid = 0x1a4
mtime = Tue Apr 12 18:18:29 CST 2016
pZxid = 0x1a1
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 9
numChildren = 0
```

# 删除节点

```java
[zk: localhost:2181(CONNECTED) 4] ls /     
[demo, config-center, config, zookeeper]
[zk: localhost:2181(CONNECTED) 5] delete /demo
[zk: localhost:2181(CONNECTED) 6] ls /        
[config-center, config, zookeeper]
```