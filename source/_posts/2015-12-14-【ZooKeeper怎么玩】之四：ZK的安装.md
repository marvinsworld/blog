title: 【ZooKeeper怎么玩】之四：ZK的部署
date: 2015-12-14 21:18:28
categories: ZooKeeper怎么玩
tags:
---
<img src="/img/zk_install.png" width="250" height="250" class="img-topic" />
Zookeeper的部署比较简单,不过仍然要注意一些细节,尤其是在生产环境中部署时更多的需要考虑硬件条件,优化及性能方面的问题.当然这里只是一些个人的见解,如有不妥,请各位看官带着辩证的思想思考.
<!--more-->

# 伪集群部署



# 集群部署

首先需要准备<strong>至少三台</strong>机器来搭建zookeeper集群,为什么至少三台服务器呢?参见[为什么推荐部署Zookeeper的机器是奇数呢](#为什么推荐部署Zookeeper的机器是奇数呢 "为什么"),这里咱们专注如何集群部署.

1. 安装JDK,准备java运行环境.Zookeeper是由java语言编写的,版本JDK6或者更高.
2. 下载Zookeeper安装包.推荐使用stable稳定版本.解压指定的目录.
3. 配置zoo.cfg
```java
	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/data/zookeeper
	clientPort=2181
	server.1=zoo1:2888:3888
	server.2=zoo2:2888:3888
	server.3=zoo3:2888:3888
```
	该配置所在的位置是%ZK_HOME%/conf目录下,参数不做详细解释.不过需要注意的是:
	* 第4行,dataDir属性强烈不推荐使用默认的tmp,在使用过程中由于系统会清空tmp目录,造成一些奇特现象.
	* 第5行,clientPort是客户端端口
	* 第6行,server.id=host:port:port中的id是唯一的,范围在1~255,第一个port是从follower节点连接到leader节点的端口;第二个port是进行leader选举的端口
4. 拷贝步骤3的配置到其他服务器上.
5. 启动服务器
	到%ZK_HOME%/bin目录下,使用如下命令启动服务器:
```java
	zkServer.sh start
```	
	启动客户端的命令是:
```java
	zkCli.sh
```		

# 为什么推荐部署Zookeeper的机器是奇数呢

官网强烈建议部署zookeeper机器的数量是奇数的,原因在于zookeeper的选举机制,选票超过半数的机器才有可能成为Leader,因此当整个集群中只有两台服务器或者整个集群超过半数的机器都挂掉时,是无法选举出Leader的.其实这是一个集群资源利用最大化的问题,例如集群中有三个节点,允许挂掉一个节点,这时还剩余两个节点,超过半数;而集群中有四个节点时,最多也只能挂掉一个节点,因为如果再挂掉一个,整个集群就剩下两个节点,无法满足超过集群数的半数这个条件.综合来看两者的容灾能力是一样的.so,你懂的

# 系统要求

操作系统上,Zookeeper对于主流的操作系统都是支持的,比如GNU/Linux,Sun Solaris,FreeBSD,MacOSX,Win32,Win64等(好像主流的操作系统也就这些了呵呵).软件上,Zookeeper是由java语言编写的,需要至少<strong>使用JDK6或者更高的版本</strong>.FreeBSD操作系统的话,需要openjdk7.

Zookeeper的定位就是集群,因此在学习研究推荐使用集群部署的方式,不过考虑到平时开发环境中服务器资源有限,也可以使用单机部署,学习更加方便.

# 单机部署

单机部署更是简单,只需要把集群部署中第3步的配置修改如下配置,启动即可,so easy!
```java
	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/data/zookeeper
	clientPort=2181
```	
# 结束

总体来说,zookeeper的部署很简单,在研究学习过程中,还是推荐大家使用伪集群模式部署.在生产环境使用zookeeper时还需要根据硬件条件调整zookeeper的相应配置
