title: Basic Paxos算法
date: 2016-06-15 10:03:09
categories: 分布式算法
tags: [一致性,分布式]
---
<img src="/img/paxos.jpg" width="600" height="150" class="img-topic" />
Paxos算法是一种提高分布式系统容错性的一致性算法，主要为解决冗余副本一致性的问题。
<!--more-->

# 角色
## Server端
**Proposer（提议者）**：将客户端发送的请求向Acceptor提案，通过二阶段方式尝试让Acceptor接受（accept）提案，在发生冲突后，尝试与Acceptor沟通后按照以Acceptor为准的方式协调冲突。
**Acceptor（决策者）**：根据“喜新厌旧”的原则接收（receive）信物，并承诺只接受（accept）更大编号的提案。
**Learner（获知者）**：接收Acceptor已接受（accept）的提案，发现超过额定数的Acceptor接受了同一个提案，形成决议（chosen）通知其他Learner，响应客户（client）。

## Client端
**Client（客户）**：向Server端发送请求，等待回应。比如：向分布式系统发送写请求。

# Paxos模型
![Paxos模型](/img/paxos-model.png "Paxos模型")
> 上图是Paxos所抽象出的模型，为了简化说明，省略了Server1和Server3中的一些细节，这里拿Server2为例来进行说明。每个Server中都抽象出了Proposer, Acceptor和Learner三个角色。每个OP被抽象成一个Proposal，Proposer用来发起Proposal, Acceptor用来决策是否接受Proposal, Learner用来获取各Acceptor决策的结果。与之对应，Paxos算法也分为三个过程：Prepare(准备发起Proposal, 图中绿线), Accept(发起Proposal并协商接受, 图中绿线), Learn(学习获取接受的Proposal, 图中蓝线)。引自[[6](#)(http://www.cnblogs.com/bodhitree/p/5065754.html)]

# 名词解释
**额定数（Quorum）**：字面意思是选举法定人数，在这里表示提案被接受（accept）的Acceptor的数量
**提案（Proposal）**：{epochNo，Value}元组
**提案编号（epochNo）**:由Proposer生成，全局唯一且自增
**提案内容（Value）**：转发Client的请求
**决议（Chosen Value）**：提案被陪审团Learner（Jury Learner）批准（chosen）后的提案

# N种动作
## Acceptor动作
**承诺（promise）**：承诺不再接受比当前持久化的epochNo小的提案
**接受（accept）**：接受Proposer的提案
**通知（notify）**：接受（accept）提案后，通知Jury Learner
**拒绝（reject）**：拒绝当前Proposer的提案，告知Proposer停止尝试当前提案

## Proposer动作
**create（生成）**：生成待发起提案的编号
**prepare（申请）**：向Acceptor申请发起提案的权限
**check（自检）**：根据收到多数向Acceptor的prepare后的响应，决定是重新发起prepare还是commit
**commit（提交）**：根据Acceptor的promise响应，提交提案

## Learner动作
**获知（learn）**接收Acceptor的通知
**发现（discover）**：发现多数Acceptor接受（accept）提案
**批准（chosen）**：发现后，形成决议
**广播（broadcast）**：将决议广播其他Learner已经批准的的决议
**响应（response）**：响应客户（client）

## Client动作
**请求（request）**：向Server发起请求