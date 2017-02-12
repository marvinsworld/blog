title: Basic Paxos算法
date: 2016-06-15 10:03:09
categories: 分布式
tags: [一致性,分布式]
---
<img src="/img/paxos.jpg" width="600" height="150" class="img-topic" />
Paxos算法是一种分布式系统高容错性的一致性算法，为了解决冗余副本间数据一致性的问题。Paxos本身不是很复杂，只是Lamport描述的过于晦涩。从Paxos论文发表到最终被大家认可经历了十多年的时间，如果作者Lamport能够一开始就换一种表达方式的话，或许Paxos在分布式系统中的应用能够更进一步。
<!--more-->

# Paxos特性
在分布式系统中，一般会采用冗余副本的形式，保证高可用，防止因为宕机、网络等异常而造成系统的不可用。**Paxos可以在分布式异常环境中（包括不超过额定数的节点宕机、网络异常等造成的消息延迟、丢失、重发等异常情况），快速正确地对某个数据的值达成一致，保证不论发生以上任何异常，都不会破坏决议的一致性。**

# Paxos解决的问题
我们先把Paxos放在一边，把视野放大到分布式环境中如何解决冗余副本上数据一致性的问题。
![操作序列](/img/serilied-consistency.png "操作序列")

上图中，C1是一个客户端，N1、N2、N3是分布式部署的三个服务器，初始状态下N1、N2、N3三个服务器中某个数据的状态都是S0。当客户端要向服务器请求处理操作序列：op1op2op3时（op表示operation）（这里把客户端的写操作简化成向所有服务器发送相同的请操作序列，实际上可能通过Master/Slave模式处理）。如果想保证在处理完客户端的请求之后，N1、N2、N3三个服务器中的数据状态都能从S0变成S1并且一致的话（或者没有执行成功，还是S0状态），就要保证N1、N2、N3在接收并处理操作序列op1op2op3时，严格按照规定的顺序正确执行opi，要么全部执行成功，要不就全部都不执行。

所以，针对上面的场景，paxos解决的问题就是如何依次确定不可变操作opi的取值，也就是确定第i个操作什么，在确定了opi的内容之后，就可以让各个副本执行opi操作。引自[[8](http://www.hollischuang.com/archives/693)]

Paxos真正要解决的问题，Wiki上是这样描述的：
> 在一个分布式数据库系统中，如果各节点的初始状态一致，每个节点都执行相同的操作序列，那么他们最后能得到一个一致的状态。为保证每个节点执行相同的命令序列，需要在每一条指令上执行一个“一致性算法”以保证每个节点看到的指令一致

# 为什么选择Paxos
解决一致性问题的方法有哪些？例子（暂略）

# 角色
## Server端
- **Proposer（提议者）**：将客户端发送的请求向Acceptor提案，通过二阶段方式尝试让Acceptor接受（accept）提案，在发生冲突后，尝试与Acceptor沟通后按照以Acceptor为准的方式协调冲突。
- **Acceptor（决策者）**：根据“喜新厌旧”的原则接收（receive）epochNo，并承诺只接受（accept）更大编号的提案。
- **Learner（获知者）**：接收Acceptor已接受（accept）的提案，发现超过额定数的Acceptor接受了同一个提案，形成决议（chosen）通知其他Learner，响应客户（client）。

## Client端
- **Client（客户）**：向Server端发送请求，等待回应。比如：向分布式系统发送写请求。

# Paxos模型
![Paxos模型](/img/paxos-model.png "Paxos模型")
> 上图是Paxos所抽象出的模型，为了简化说明，省略了Server1和Server3中的一些细节，这里拿Server2为例来进行说明。每个Server中都抽象出了Proposer, Acceptor和Learner三个角色。每个OP被抽象成一个Proposal，Proposer用来发起Proposal, Acceptor用来决策是否接受Proposal, Learner用来获取各Acceptor决策的结果。与之对应，Paxos算法也分为三个过程：Prepare(准备发起Proposal, 图中绿线), Accept(发起Proposal并协商接受, 图中绿线), Learn(学习获取接受的Proposal, 图中蓝线)。引自[[6](http://www.cnblogs.com/bodhitree/p/5065754.html)]

# 名词解释
- **额定数（Quorum）**：字面意思是选举法定人数，在这里表示提案被接受（accept）的Acceptor的数量
- **提案（Proposal）**：{epochNo，Value}元组
- **提案编号（epochNo）**:由Proposer生成，全局唯一且自增
- **提案内容（Value）**：转发Client的请求
- **决议（Chosen Value）**：提案被陪审团Learner（Jury Learner）批准（chosen）后的提案

# N种动作
## Acceptor动作
- **承诺（promise）**：承诺不再接受比当前持久化的epochNo小的提案
- **接受（accept）**：接受Proposer的提案
- **通知（notify）**：接受（accept）提案后，通知Jury Learner
- **拒绝（reject）**：拒绝当前Proposer的提案，告知Proposer停止尝试当前提案

## Proposer动作
- **create（生成）**：生成待发起提案的编号
- **prepare（申请）**：向Acceptor申请发起提案的权限
- **check（自检）**：根据收到多数向Acceptor的prepare后的响应，决定是重新发起prepare还是commit
- **commit（提交）**：根据Acceptor的promise响应，提交提案

## Learner动作
- **获知（learn）**接收Acceptor的通知
- **发现（discover）**：发现多数Acceptor接受（accept）提案
- **批准（chosen）**：发现后，形成决议
- **广播（broadcast）**：将决议广播其他Learner已经批准的的决议
- **响应（response）**：响应客户（client）

## Client动作
- **请求（request）**：向Server发起请求

# 算法三阶段
## Phase1:Prepare
**角色Proposer**
**P1a**：Proposer生成了一个编号epochNo（规则：全局唯一且递增），向Acceptor半数以上成员发送Prepare请求。

**角色Acceptor**
**P1b**：Acceptor收到Prepare请求，判断：收到的epochNo是否之前已回复的最大提案的epochNo大,比对结果是true：
> 1. 持久化epochNo，标记为Max-epochNo（下图中的#1）
> 2. 回复请求，返回已经**Accept**的提案中**最大的**epochNo和对应的value（下图中{NULL,NULL}）
> 3. 承诺，不会Accept任何小于Max-epochNo的提案

![Prepare阶段](/img/paxos-prepare.png "Prepare阶段")

## Phase2:Accept
**角色Proposer**
**P2a**：Proposer收到一部分Acceptor的prepare请求后的回复，情况如下：
> 1. 回复数量大于一半的Acceptor的数量，且所有回复的value都是空，则Proposer发送accept请求，带上**自己指定的value**
> 2. 回复数量大于一半的Acceptor的数量，且有的回复value不为空，则Proposer发送accept请求，带上**回复中epochNo最大的value**
> 3. 回复数量小于等于一半Acceptor数量，则尝试生成更大的epochNo，再执行P1a

**角色Acceptor**
**P2b**：Acceptor收到accept请求后，
> 1. 收到的epochNo等于Max-epochNo，回复提交成功，持久化epochNo和value
> 2. 收到epochNo小于Max-epochNo，则不回复。P1b中3承诺

![Accept阶段](/img/paxos-accept.png "Accept阶段")

## Phase3:Chosen
**角色Acceptor**
**P3a**：Acceptor将已经accept的请求通知给陪审团Learner（Jury Learner）

![Chosen阶段](/img/paxos-chosen.png "Chosen阶段")

**角色Jury Learner**
**P3b**：陪审团Learner（Jury Learner）收到一部分Acceptor回复accept请求的结果。
> 1. 回复数量大于一半的Acceptor数量，表示value提交成功，可以发送广播给所有的Proposer、Acceptor、Learner，通知它们已chosen的value
> 2. 回复数量小于等于一半的Acceptor数量，Proposer会尝试生成更大的epochNo，重复P1a操作。

![Broadcast阶段](/img/paxos-broadcast.png "Broadcast阶段")

# 疑问
1. Paxos究竟在解决一个什么问题？
> 分布式存储系统中存在冗余副本，如果不对更新操作的顺序进行控制，就无法保证每个副本的更新操作顺序是一致的，进而无法保证副本之间数据一致性。Paxos旨在解决多副本之间如何保证一次更新操作的数据一致性问题。如果要解决多次更新操作的问题，需要Multi Paxos。

2. 第一阶段Prepare在做什么？
> 如果同时有多个议案，可能会导致每个acceptor都接受一个议案，形成不了大多数，造成死锁。第一阶段是竞争提案号，Prepare阶段采用“后来居上”的方式，后者在有更大的epoch的情况下，可以获取互斥访问权限，不会造成死锁。

3. 第二阶段Accept在做什么？
> 某些Proposer依次或者同时发起不同的提案，由于网络故障，提案的请求可能在中途部分丢失，每种提案都可能都只有不同少数派Acceptor接收到，每个提案都无法形成多数派，形成死锁。第二阶段是竞争提案值，Accept阶段采用“先到先得”的方式，

4. 怎么理解accept？
> 当Acceptor接收（receive）到Proposer提交的请求，经过判断后发现这次请求中带着的提案编号是当前Acceptor持久化的提案编号，确定当前的value值为选定的value值，这是Acceptor接受（accept）请求

5. 谁来批准提案，形成决议的时机是什么时候？
> 在优化的方案中，有这样一组陪审团Learners（暂且称为Jury Learners），接收（receive）Acceptor已接受（accept）的提案，当这组Learners发现同一个提案被多数Acceptor接受（accept），就会形成（chosen）决议。

6. 谁负责通知Learner？
> 由陪审团Learners（Jury Learners）发起广播，告知其他Learner决议已经形成，同步决议。

7. 每个副本如何生成唯一且不重复的epoch？
> Google的Chubby论文给出了一种步长的方式:以3个proposer P1、P2、P3为例，开始m=0,编号分别为0，1，2P1提交的时候发现了P2已经提交，P2编号为1 \> P1的0，因此P1重新计算编号：new P1 = 1\*3+0 = 4，P3以编号2提交，发现小于P1的4，因此P3重新编号：new P3 = 1\*3+2 = 5

8. paxos算法中epoch编号一直递增不重复的，请问在实现paxos算法时，如何处理溢出的问题呢？可能出现proposer的epoch溢出，而acceptor的latest\_prepared\_epoch是一个很大的值，从而算法无法进行下去？
> Epoch不一定要是int整数，只要是可以比较大小的对象就可以。因此解决溢出的方法也很多，例如直接用数组模拟大数，溢出后增加数组的元素数等等。

# 参考
1. [Paxos算法1-算法形成理论](http://blog.csdn.net/chen77716/article/details/6166675)
2. [Paxos算法2-算法过程](http://blog.csdn.net/chen77716/article/details/6170235)
3. [Paxos算法3-实现探讨](http://blog.csdn.net/chen77716/article/details/6172392)
4. [对 Paxos 的一些理解](https://yq.aliyun.com/articles/53351)
5. [Paxos算法深入分析](http://blog.csdn.net/anderscloud/article/details/7175209)
6. [理解 Paxos](http://www.cnblogs.com/bodhitree/p/5065754.html)
7. [分布式一致性算法——paxos](http://www.hollischuang.com/archives/693)
8. [2PC到3PC到Paxos到Raft到ISR](https://segmentfault.com/a/1190000004474543)
9. [Paxos算法小结](http://www.cnblogs.com/shaohef/p/4499881.html)
10. [Paxos/Mutil-paxos 算法浅析](http://www.chongh.wiki/blog/2016/04/27/paxos-mutilpaxos/)
11. [[Paxos三部曲之一] 使用Basic-Paxos协议的日志同步与恢复](http://oceanbase.org.cn/?p=90#rd&sukey=3903d1d3b699c208f8fd9337d59ef4b09ab31499f39398e7321ec8f79d2ff57fd6062dc4dc9b838183e81762efbba0a6)
12. [一致性算法 Paxos Raft 的一些整理](http://wangzhezhe.github.io/blog/2015/10/05/consensus-paxos-and-raft/)
13. [Paxos协议 学习小结](http://baozh.github.io/2016-03/paxos-learning/)