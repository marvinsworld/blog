title: 【ZooKeeper怎么玩】之四：ZK的部署
date: 2015-12-14 21:18:28
categories: ZooKeeper怎么玩
tags:
---
<img src="/img/zk_install.png" width="250" height="250" class="img-topic" />
Zookeeper的部署比较简单,不过仍然要注意一些细节,尤其是在生产环境中部署时更多的需要考虑硬件条件,优化及性能方面的问题.当然这里只是一些个人的见解,如有不妥,请各位看官带着辩证的思想思考.
<!--more-->

# 系统要求

操作系统上,Zookeeper对于主流的操作系统都是支持的,比如GNU/Linux,Sun Solaris,FreeBSD,MacOSX,Win32,Win64等(好像主流的操作系统也就这些了呵呵).软件上,Zookeeper是由java语言编写的,需要至少<strong>使用JDK6或者更高的版本</strong>.FreeBSD操作系统的话,需要openjdk7.

Zookeeper的定位就是集群,因此在学习研究推荐使用集群部署的方式,不过考虑到平时开发测试环境中资源有限的情况,也可以使用单机部署,调试更加方便.

#单机部署