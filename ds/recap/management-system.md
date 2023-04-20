
# Zookeeper

## 背景

在传统的HDFS1.0中，单点故障是使用SecondaryNameNode，主要用途：

- 冷备份
- 主要是防止日志文件过大，导致NameNode节点恢复时时间过长

使用Zookeeper做到高可用

- NameNode之间的数据同步
- NameNode之间的状态感知（一旦Active出现故障，立即切换Standby）

在配置多个NameNode时候，谁会成为Active？

- 由于在动态变化，不能写入配置文件中

## 体系架构

ZooKeeper：轻量级的分布式系统 

作用：用于解决分布式应用中通用的协作问题 

- 命名管理 Naming
- 配置管理 Configuration Management 
- 集群管理 Group Membership （监控节点的状态）
- 同步管理 Synchronization 

角色

- server：负责管理元数据
  - leader：某一个server，保证分布式数据一致性的关键，何时需要选取leader？
    - 服务器初始化启动
    - 服务器运行期间无法和Leader保持连接
- client：需要进行协作的用户（系统）
  - 提供元数据
  - 获取元数据，并执行相关操作

## 数据模型

- Znode
  - 不保存数据，保存元数据或配置信息
  - 可以保存时间戳进行版本控制
  - 类型：常规/临时
  - 标识：sequential flag
- client
  - 可以在某个Znode上设置一个Watcher，来检测该Znode上的变化
- server
  - 当这个Znode中的存储数据修改/子节点目录变化，可以通知设置监控的客户端

## Znode

- 分为四种类型
  - 是否会自动删除
    - 常规/临时（在Session结束后自动删除）
    - **临时节点不能有子节点目录**
  - 是否带有顺序号
- Znode是有版本的，每个Znode中存储的数据可以有多个版本，也就是一个访问路径中可以存储多份数据

## 应用

1. 命名管理
   -  统一命名服务:分布式应用通常需要一套命名规则，既能够产生唯一的名称又便于人识别和记住（如访问分布式中的各个节点）
2. 动态配置管理
   - 如用户命令行修改了默认的参数（master节点IP，worker节点数量，webport）
   - 配置信息存在 ZooKeeper 某个目录节点，所有机器watch该目录节点 
   - 一旦发生变化，每台机器收到通知，然后从 ZooKeeper获取新的配置信息应用到系统中 
3. 集群管理
   - 动态感知机器增减
     - 创建**临时文件**，当新建文件时，个数发生变化，减少文件时，由于是临时的，文件数减少
   - **选主**
     - 注意这里的选主与zookeeper本身选主（基于一致性协议）的区别
     - 创建一个**临时有序**的文件，每次选择序号最小的
4. 共享锁
   - 若所有节点都监控一个Znode，则Zookeeper需要发出大量的事件通知
     - 每次lock都需要发出很多通知，但只有一个能拿到锁，代价很大
     - **羊群效应**
   - 进行分组处理
     - 对每个需要拿锁的client创建一个**临时有序的Znode**，每次取序号最小的
     - 若自己是最小的，则拿到锁，否则监控比自己序号小的一个Znode
     - 可以实现**负载均衡**
5. 队列管理
   - 同步队列
   - **双屏障**：允许客户端在计算的开始和结束时同步。当足够的进程加入到双屏障时，进程开始计算。当计算完成后，离开屏障（BSP model中使用）。
     - 使用ready和process节点实现
       - 每个进程到ready中注册，达到要求后到process中建立节点
     - 为什么需要两个节点
       1. 若只有ready，迭代一次删除，但循环没有终止，还需要注册
       2. 若在做时有新节点加入，则不知道什么时候结束（本来的判断标识为集合为空）

# YARN

在MapReduce系统中，`JobTracker`需要进行资源管理和任务调度

- 存在单点故障的风险
- 内存开销大
- 资源分配只考虑MapReduce任务数量，不考虑任务需要的CPU/内存，TaskTracker所在节点容易内存溢出
- 资源划分不灵活

资源管理不单是MapReduce系统所需要的，而是通用的

## 体系结构

- ResourceManager
  - 处理客户端请求
  - 应用程序管理器：启动/监控ApplicationMaster，监控NodeManager
  - 资源调度器：资源分配与调度
- ApplicationMaster
  - 与RM进行交互：为内存应用申请资源并分配给内部任务
  - 任务调度/监控与容错
- NodeManager
  - 单个节点上的资源管理，处理来自ApplicationMaster/ResourceManager的命令
  - **只负责与容器有关的事情，不负责具体每个任务自身的状态管理**
- Container
  - 运行任务

两个主从关系

- RM -> NodeManager 资源管理
- AM -> Container 计算，**任务实际上向AM汇报自己的状态和进度**

## 发展目标

- 实现集群资源共享和资源弹性收缩
- 提高集群利用率
- 不同计算系统可以共享底层存储，避免了数据集跨集群的移动



# KEYS

## Zookeeper

- 在配置多个NameNode时候，谁会成为Active？
- zookeeper的主要作用
- 在zookeeper中谁时client？
- Znode的四种类型
- 羊群效应
- 如何实现双屏障，什么系统需要双屏障？BSP Model

## YARN

- 体系结构中的两个主从关系
- YARN多租户部署
  - 增强集群的资源利益率
  - 减少维护代价
- YARN发展目标

