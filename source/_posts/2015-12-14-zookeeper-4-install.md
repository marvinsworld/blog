title: ZooKeeper怎么玩之四：ZK的部署
date: 2015-12-14 21:18:28
categories: ZooKeeper怎么玩
tags:
---
<img src="/img/zk_install.png" width="250" height="250" class="img-topic" />
ZooKeeper有两种部署方式:集群模式和单机模式.集群模式一般用于生产环境,对系统的可用性要求较高,不会因为单点故障而导致整个系统不可用.而单机模式是ZooKeeper的最低要求,仅供平时测试,开发时使用.
<!--more-->
# 集群部署

首先需要准备<strong>至少三台</strong>服务器来搭建ZooKeeper集群,为什么至少三台服务器呢?参见[为什么推荐部署ZooKeeper的机器是奇数呢](#为什么推荐部署ZooKeeper的机器是奇数呢 "为什么").

1. 准备java运行环境,安装JDK6或更高版本.(因为ZooKeeper是由java语言编写的)
2. 下载ZooKeeper安装包.推荐使用stable稳定版本,比如Release 3.4.6,下载后解压一个目录中,比如/usr/local/目录下.
3. 修改配置文件zoo.cfg.
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
	该配置所在的位置是<strong>%ZK_HOME%/conf</strong>目录下,参数不做详细解释.不过需要注意的是:
	* 第4行,dataDir属性强烈不推荐使用默认的tmp,在使用过程中由于操作系统会清空tmp目录,造成一些奇特现象.
	* 第5行,clientPort是ZooKeeper客户端端口.
	* 第6行,server.id=host:port:port中的id是唯一的,用来标识机器在集群中序号,范围在1~255,第一个port是从follower节点连接到leader节点的端口;第二个port是进行leader选举的端口.
4. 创建myid文件.
	在配置文件dataDir项所配置目录下创建名为myid的文件,并在该文件第一行写上该机器在集群中对应的序号,比如服务器IP是zoo1的,在对应的myid文件中写上1,标识当前服务器对应的序号.
5. 重复步骤3,4的配置到其他服务器上.
6. 启动服务器
	到%ZK_HOME%/bin目录下,使用```zkServer.sh start```启动ZooKeeper,见到如下内容则标识ZooKeeper启动成功.
```java
	JMX enabled by default
	Using config: /usr/local/zk/bin/../conf/zoo.cfg
	Starting zookeeper ... STARTED
```	

	启动客户端的命令是:
```java
	zkCli.sh
```		

# 为什么推荐部署ZooKeeper的机器是奇数呢

官网强烈建议部署zookeeper机器的数量是奇数的,原因在于ZooKeeper的选举机制,选票超过半数的机器才有可能成为Leader,因此当整个集群中只有两台服务器或者整个集群超过半数的机器都挂掉时,是无法选举出Leader的.其实这是一个集群资源利用最大化的问题,例如集群中有三个节点,允许挂掉一个节点,这时还剩余两个节点,超过半数;而集群中有四个节点时,最多也只能挂掉一个节点,因为如果再挂掉一个,整个集群就剩下两个节点,无法满足超过集群数的半数这个条件.综合来看两者的容灾能力是一样的.so,你懂的


# 伪集群部署

学习研究时更多的推荐大家使用伪集群模式,将需要多个硬件的要求改成在一台机器上.实现起来也很简单,把ZooKeeper的压缩包解压到三个不同的目录,将上面的配置文件,只修改server启动端口和client监听端口,ip不变.即可


# 系统要求

操作系统上,ZooKeeper对于主流的操作系统都是支持的,比如GNU/Linux,Sun Solaris,FreeBSD,MacOSX,Win32,Win64等(好像主流的操作系统也就这些了呵呵).软件上,ZooKeeper是由java语言编写的,需要至少<strong>使用JDK6或者更高的版本</strong>.FreeBSD操作系统的话,需要openjdk7.

ZooKeeper的定位就是集群,因此在学习研究推荐使用集群部署的方式,不过考虑到平时开发环境中服务器资源有限,也可以使用单机部署,学习更加方便.

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


