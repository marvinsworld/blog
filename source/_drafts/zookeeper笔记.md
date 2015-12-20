title: zookeeper笔记
tags:
---

1.如何理解namespace的概念
	相当于树的根,注册到相同namespace下的节点,其根节点时这个namespace

2.NodeCache.start()理解
	使用时应该调用start(boolean),不使用时调用close()方法
	start()的参数,默认是false,如果是true时,NodeCache会在启动监听之前调用internalRebuild()方法,读取该监听节点的数据内容,保存到NodeCache中.