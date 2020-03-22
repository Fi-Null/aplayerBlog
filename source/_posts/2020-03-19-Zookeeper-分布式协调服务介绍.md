---
title: Zookeeper 分布式协调服务介绍
date: 2020-03-19 11:18:21
tags:
---

## 分布式系统

分布式系统的简单定义：分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统。

分布式系统的特征：

- 分布性：系统中的计算机在空间上随意分布和随时变动
- 对等性：系统中的计算机是对等的，没有主从之分
- 并发性：并发性操作是非常常见的行为
- 缺乏全局时钟：系统中的计算机具有明显的分布性，且缺乏一个全局的时钟序列控制，所以很难比较两个事件的先后
- 故障总是会发生：任何在设计阶段考虑到的异常情况，一定会在系统实际运行中发生，并且还会遇到很多在设计时未考虑到的异常故障

随着分布式架构的出现，越来越多的分布式应用会面临数据一致性问题。

## 选择Zookeeper

Zookeeper是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于它实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、master选举、分布式锁和分布式队列等功能。

Zookeeper致力于提供一个高性能、高可用，具有严格的顺序访问控制能力的分布式协调服务；其主要的设计目标是简单的数据模型、可以构建集群、顺序访问、高性能。Zookeeper已经成为很多大型分布式项目譬如Hadoop、HBase、Storm、Solr等中的核心组件，用于分布式协调。

Zookeeper可以保证如下**分布式一致性特性**：

- 顺序一致性：从同一个客户端发起的事务请求，最终将会严格地按照其发起顺序被应用到Zookeeper中去
- 原子性：所有事务请求的处理结果在整个集群中所有的机器上的应用情况是一致的
- 单一视图：无论客户端连接的是哪个Zookeeper服务器，其看到的服务器数据模型都是一致的
- 可靠性：一旦服务端成功地应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会被一直保留下来，除非有另一个事务又对其进行了变更
- 实时性：在一定的时间内，客户端最终一定能够从服务端上读取到最新的数据状态

![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/architect/zkservice.png)

## Zookeeper 基本概念

- 集群角色
    - Leader：客户端提供读和写服务
    - Follower：提供读服务，所有写服务都需要转交给Leader角色，参与选举
    - Observer：提供读服务，不参与选举过程，一般是为了增强Zookeeper集群的读请求并发能力
- 会话 (session)
    - Zk的客户端与zk的服务端之间的连接
    - 通过心跳检测保持客户端连接的存活
    - 接收来自服务端的watch事件通知
    - 可以设置超时时间

#### ZNode 节点

ZNode 是Zookeeper中数据的最小单元，每个ZNode上可以保存数据(byte[]类型)，同时可以挂在子节点，因此构成了一个层次化的命名空间，我们称之为树

![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/architect/znode.png)

- 节点是有生命周期的，生命周期由**节点类型**决定：
    - 持久节点(PERSISTENT)：节点创建后就一直存在于Zookeeper服务器上，直到有删除操作主动将其删除
    - 持久顺序节点(PERSISTENT_SEQUENTIAL)：基本特性与持久节点一致，额外的特性在于Zookeeper会记录其子节点创建的先后顺序
    - 临时节点(EPHEMERAL)：声明周期与客户端的会话绑定，客户端会话失效时节点将被自动清除
    - 临时顺序节点(EPHEMERAL_SEQUENTIAL)：基本特性与临时节点一致，但添加了顺序的特性
- 每个节点都有**状态信息**，抽象为 Stat 对象，状态属性如下：
    - czxid：节点被创建时的事务ID
    - mzxid：节点最后一个被更新时的事务ID
    - ctime：节点创建时间
    - mtime：节点最后一个被更新时间
    - version：节点版本号
    - cversion：子节点版本号
    - aversion：节点的ACL版本号
    - ephemeralOwner：创建该临时节点的会话的sessionID，若为持久节点则为0
    - dataLength：数据内容长度
    - numChildren：子节点数量
- 权限控制ACL (Access Control Lists)
    - CREATE：创建子节点的权限
    - READ：获取节点数据和子节点列表的权限
    - WRITE：更新节点数据的权限
    - DELETE：删除子节点的权限
    - ADMIN：设置节点ACL的权限

#### watcher机制

Zookeeper 引入watcher机制来实现发布/订阅功能，能够让多个订阅者同时监听某一个节点对象，当这个节点对象状态发生变化时，会通知所有订阅者。

Zookeeper的watcher机制主要包括客户端线程、客户端WatchManager、Zookeeper服务器三个部分。其工作流程简单来说：客户端在向Zookeeper服务器注册Watcher的同时，会将Watcher对象存储在客户端的WatchManager中；当Zookeeper服务器端触发Watcher事件后，会向客户端发送通知，客户端线程从WatchManager中取出对应的Watcher对象来执行回调逻辑

![](https://raw.githubusercontent.com/Fi-Null/blog-pic/master/blog-pics/architect/zk_watch.png)

可以设置的两种 Watcher

- NodeCache
    - 监听数据节点的内容变更
    - 监听节点的创建，即如果指定的节点不存在，则节点创建后，会触发这个监听
- PathChildrenCache
    - 监听指定节点的子节点变化情况
    - 包括新增子节点、子节点数据变更和子节点删除

**Zookeeper机制的特点：**

1. 一次性触发数据发生改变时，一个watcher event会被发送到client，但是client***只会收到一次这样的信息\***。
2. watcher event异步发送watcher的通知事件从server发送到client是***异步\***的，这就存在一个问题，不同的客户端和服务器之间通过socket进行通信，由于***网络延迟或其他因素导致客户端在不通的时刻监听到事件\***，由于Zookeeper本身提供了***ordering guarantee，即客户端监听事件后，才会感知它所监视znode发生了变化\***。所以我们使用Zookeeper不能期望能够监控到节点每次的变化。Zookeeper***只能保证最终的一致性，而无法保证强一致性\***。
3. 数据监视Zookeeper有数据监视和子数据监视getdata() and exists()设置数据监视，getchildren()设置了子节点监视。
4. 注册watcher ***getData、exists、getChildren\***
5. 触发watcher ***create、delete、setData\***
6. ***setData()\***会触发znode上设置的data watch（如果set成功的话）。一个成功的***create()\*** 操作会触发被创建的znode上的数据watch，以及其父节点上的child watch。而一个成功的***delete()\***操作将会同时触发一个znode的data watch和child watch（因为这样就没有子节点了），同时也会触发其父节点的child watch。
7. 当一个客户端***连接到一个新的服务器上\***时，watch将会被以任意会话事件触发。当***与一个服务器失去连接\***的时候，是无法接收到watch的。而当client***重新连接\***时，如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。只有在一个特殊情况下，***watch可能会丢失\***：对于一个未创建的znode的exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个watch事件可能会被丢失。
8. Watch是轻量级的，其实就是本地JVM的***Callback\***，服务器端只是存了是否有设置了Watcher的布尔类型

## Zookeeper 的典型应用场景

Zookeeper 是一个典型的发布/订阅模式的分布式数据管理与协调框架，开发人员可以使用它来进行分布式数据的发布和订阅。

通过对 Zookeeper 中丰富的数据节点进行交叉使用，配合 Watcher 事件通知机制，可以非常方便的构建一系列分布式应用中年都会涉及的核心功能，如：

1. 数据发布/订阅
2. 负载均衡
3. 命名服务
4. 分布式协调/通知
5. 集群管理
6. Master 选举
7. 分布式锁
8. 分布式队列