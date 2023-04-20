# 分布式文件系统

## 文件和目录

- 文件系统的主要优点是实现`按名存取`
- 如何实现？
  - 文件目录
- 文件的物理结构
  - 顺序文件/连续存储
  - 索引文件
  - 链接文件

## 分布式文件系统

### 实现思路

1. 保证每台机器可以透明的访问其他机器上的文件（P2P结构）
2. 将所有机器的文件系统关联起来，形成一个对外统一的整体（主从结构）
   - 通过一致性哈希/RPC实现

### 文件访问

主要使用`Immutable files`，不能修改

### 备份与一致性

- 客户端备份

  - 客户端得到文件的副本，修改后返回给服务端

- 服务端备份

## HDFS

### 假设

1. 硬件的异常比软件的异常更加常见
2. 应用程序关注的是吞吐量，而不是响应时间
3. 存储的数据较大
4. 文件仅允许追加，不允许修改
5. 移动计算比移动数据更划算
6. **采用流式数据访问（数据批处理）**

### 数据块

**与操作系统中block的区别**

- block是操作系统中读取的基本单位（总是一样大的），目的是为了省IO
- 在HDFS中是为了切分文件（最后的数据库可能较小）

### 体系架构

- NameNode
  - 负责文件系统元数据操作，数据块的复制和定位
  - 元数据镜像文件/日志文件，**保存在内存中**
  - 处理控制流
- SecondNameNode
  - NameNode的备份节点
  - 检查点备份，不是热备份
- DataNode
  - 集群中的每个节点一个数据节点
  - 负责数据块的存储，提供实际文件数据，**保存在磁盘中**
  - 处理数据流
  - 为什么不需要备份DataNode？

### 文件访问模型

**一次写入多次读写**

- 对于单文件，不支持并发写，只支持并发读（**无需文件锁，简单**）
- 修改内容需删除，重新写入
- 仅允许追加（对文件增加一个block），且同时只能有一个进程操作

### 数据备份

为什么需要备份：

1. 实现容错，保证数据可靠性
2. 加快数据传输速度
3. 实现负载均衡
4. 容易检查数据错误

**写入成功的备份是强一致的**

### 容错机制

- DataNode故障
  - 宕机节点上面的所有数据都会被标记为 “不可读”
  - 数据块自动复制到剩余的节点以保证满足备份因子，如默认为3
- NameNode故障
  - 根据 SecondaryNameNode 中的 FsImage 和 Editlog 数据进行恢复

# MapReduce

## MPI

是一个信息传递应用程序接口，包括协议和语义说明

已经有MPI成熟的并行计算框架，为什么还需要MapReduce？

- 传统并行计算框**架容错性差**，只能使用昂贵机器
- 编程学习很难
- 使用场景为计算密集型

## 系统架构

- `Client`
  - `提交作业`：用户编写的MapReduce程序通过Client提交到**JobTracker端**
  - `作业监控`：用户可通过Client提供的一些接口查看作业运行状态
- `JobTracker`
  - `资源管理`：监控TaskTracker与Job的状况。一旦发现失败，就将Task转移到其它节点
  - `作业调度`：将Job拆分成Task，跟踪Task的执行进度、资源使用量等信息，由TaskScheduler调度（不是一个单独的进程，是一个模块）
- `TaskTracker`
  - `执行操作`：接收JobTracker发送过来的命令并执行（如启动新Task、杀死Task等）
  - `划分资源`：使用“slot”等量划分本节点上的资源量（CPU、内存等），一个Task 获取到一个slot 后才有机会运行
  - `汇报信息`：通过**“心跳”**将本节点上**资源使用情况和任务运行进度**汇报给JobTracker
  - 将Jar包发送到TaskTracker，利用反射和代理动态加载代码
  - `Map slot`-> `Map Task`
  - `Reduce slot` -> `Reduce Task`
- `Task`
  - 任务执行
    - `Map task`
    - `Reduce task`

## 工作流程

**只有mapper完成后Reducer才可以开始：指的是一个节点完成后，才可以和被Reducer读取，并不是所有map完成。**

- `slot`
  - 在TaskTracker中，使用Slot划分本节点的资源（CPU，内存），一个Task获取到一个slot后才有机会运行
  - Map slot -> Map Task
  - Reduce slot -> Reduce Task
- TaskTracker如何汇报本节点的资源使用情况？
  - 通过心跳将资源使用和任务运行进度汇报给JobTracker
- Task如何执行Map和Reduce函数？
  - Jar包发送到TaskTracker，利用反射和代理机制动态加载代码
- 数据交换
  - 不同Map、Reduce任务之间不会发生任何信息交换
  - **有多少个Reduce，就有多少个输出**
  - 所有的数据交换都是MapReduce框架自身去实现的（shuffle）
- 如何确定shuffle到哪个reduce？
  - 通过hash partition
- `split`
  - 在`InputFormat`中将数据划分为多个**逻辑单元**，而block是物理概念
  - Hadoop为每个split创建一个map任务，split的多少决定了map任务的数目（框架决定）
  - 如何划分
    - 如果map task的物理机上有需要的split文件，则采取就近原则。

### Shuffle过程

分为Map端和Reduce端（阻塞式的，reduce一定在shuffle后结束）

#### Map端

- 溢写
  - 分区
  - 排序
    - 使其局部有序
    - 使得reduce可以直接归并，提高效率
  - 合并（Combine）
    - combine函数，用户定义，不一定需要
    - 可以数据压缩，提高效率
  - 一定溢写到磁盘（为了容错）
- 文件归并（merge）
  - 归并到磁盘（为了容错，与溢写同时做）
  - 与合并的区别
    - 归并是将key相同的记录拼接在一起，变为list
    - 合并是按照用户指定的函数进行数据合并

#### Reduce端

- copy
   - **Reduce会主动来拉取写到磁盘的数据**
   - 将读来的文件进行归并
- merge sort
   - Map的输出数据已经是有序的，Merge进行一次合并排序，一般Reduce是一边copy一边sort（磁盘和内存同时使用）

## 容错

### Task容错

- Map Task失败
  - 去HDFS重新读入数据
- Reduce Task失败
  - 从Map的中间结果中读入数据
  - 因为要写入磁盘，因此没办法实时流计算

### JobTracker失败

- 最严重的失败，Hadoop没有处理其失败的机制，是单点故障
- 所有任务需要重新运行

## 缺点

1. 输入输出/shuffle中间结果要落磁盘，磁盘IO开销大
2. 表达能力有限，只有map和reduce函数
3. 延迟高（多个job）
   - 在有依赖关系中，多个job之间的衔接涉及IO开销
   - 无依赖关系中，在前一个job执行完成之前，其他job依然无法开始（不利于并行）

## 编程

1. 单个mapreduce
2. 组合式mapreduce
   - 隐式依赖描述/显式
   - 显式依赖描述更好，因为可以让系统知道调度信息，可进行优化（短作业优先），且可以避免上个程序运行失败导致的出错
3. 链式mapreduce
   - 只有一个reduce，有多个map（在reduce前后都可以）
4. 迭代mapreduce

hadoop streaming

- 它允许用户使用任何可执行文件或者脚本文件作为Mapper和Reducer

# Spark

## 改进

1. 表达能力有限 
   - 增加join等更多复杂的函数，可以串联为**DAG**
   - 编程模型更灵活
2. 磁盘IO开销大(单个job) 
   - **非shuffle阶段避免中间结果写磁盘**
   - 迭代运算效率更高
3. 延迟高(多个job作为一整个job) 
   - **将原来的多个job作为一个job的多个阶段**
     - 有依赖关系:各个阶段之间的衔接尽量写内存
     - 无依赖关系:多个阶段可以同时执行 

## RDD抽象

弹性分布式数据集，这里的弹性主要指容错

- 是分布式内存中的一个抽象概念，提供了一种高度受限的共享式内存模型
- 具有可恢复的容错特性
- 每个RDD可分成多个分区，一个RDD的不同分区可以保存到集群中的不同节点上
- 每个分区都是一个数据集片段

### 特性

- 只读（immutable）
  - 本质上是一个只读的对象集合，不能直接修改，只能基于稳定的物理存储中的数据集创建RDD
- 支持运算操作
  - 转换（transformation）：描述RDD的转换逻辑
  - 动作（action）：标志转换结束，**触发DAG生成**
    - **惰性求值：只有遇到action操作时，才会发生真正的计算**

### RDD Lineage

指DAG的拓扑结构

- RDD经过一系列的transformation操作，产生不同的RDD，最后一个RDD经过action进行转换，并输出
- 保留RDD lineage的信息，为了容错
  - RDD依赖关系：
    - 宽依赖：存在一个父RDD的一个分区对应一个子RDD的多个分区，容错需要使用整个父RDD，将子RDD重新算一遍
    - 窄依赖：一个父RDD的分区对应于一个子RDD的分区或多个父RDD的分区对应于一个子RDD的分区，容错仅需要部分父RDD将丢失部分的子RDD计算一遍

为什么关心依赖关系？

- 分析各个RDD的偏序关系生成DAG，再通过分析各个RDD中的分区之间的依赖关系来决定**如何划分Stage** 
- 具体划分方法
  - 在DAG中进行**反向解析**，遇到宽依赖就断开 
  - 遇到窄依赖就把当前的RDD加入到Stage中 
  - **将窄依赖尽量划分在同一个Stage中，可以实现流水线计算 pipeline** 

### stage 类型

- ShuffleMapStage
  - 不是最终的stage，输出一定经过shuffle过程，并作为后续stage的输入
- ResultStage
  - 输出直接产生结果或存储，在一个job最终必定含有该类型

## 体系结构

driver -> spark master -> spark worker -> executor -> task

- `Master`负责 集群资源管理（也可以使用yarn）
- `Worker` 运行作业的工作节点，负责执行的**进程（executor）和线程（task）**

作业与任务

- Job -> DAG
  - 包含多个RDD及对应RDD的各种操作
- stage -> TaskSet
  - 一个job会分成多组task，每组task被称为stage
  - 是job的基本调度单位，代表了一组关联的，相互没有shuffle依赖关系的任务组成的任务集
- RDD -> task
  - 表示运行在executor上的工作单元

## 运行流程

1. Driver向集群管理器申请资源 
2. 集群管理器启动Executor 
3. Driver向Executor发送应用程序代码和文件
4. Executor上执行Task，运行结束后，执行结果 会返回给Driver，或写到HDFS等

## 运行特点

- Application有专属的Executor进程，并且该进程在Application运行期间一直驻留
- Executor进程以多线程的方式运行Task
- **Spark运行过程与资源管理器无关**，只要 够获取Executor进程并保持通信即可
- Task采用了**数据本地性和推测执行**等优化机制
  - 推测执行将较慢的任务再次在其它节点启动

executor相比mapreduce的优点：

1. 利用多线程来执行具体任务，减少任务的启动开销
2. 将内存和磁盘共同作为存储设备，减少IO开销

## Spark Shuffle

在stage之间是shuffle

在stage内部是流水线pipeline

- `consolidateFiles`
  - 若设置为true，则表示将一个核上的多个小文件变为大文件进行传输，提高效率
- spark pipeline 可以在内存中，**shuffle必须要压磁盘**

## Spark的主要改进

1. 算子的扩充（join）
2. 避免stage内部的shuffle写磁盘
3. 迭代时，每一次不必写磁盘，只是RDD的转换

## 容错

Master故障：没有办法（zookeeper）

Worker故障（因为Task已经是线程，所以不需要容错处理）

- Lineage机制
  - 重新计算丢失分区（宽依赖/窄依赖），无需回滚系统
  - 重算过程在不同节点之间并行，只记录粗粒度的操作
- RDD存储机制
  - RDD提供持久化接口
  - 没有标记持久化的RDD会被garbage collection回收
- 检查点机制
  - Lineage可能非常长
  - RDD存储机制主要面向本地磁盘的存储
  - 检查点机制将RDD写入可靠的外部分布式文件系统，例如HDFS
    - 只写入指定的RDD

## SQL

- Shark & Spark SQL
- RDD的局限性
  - 对于对象内部结构，即数据schema不可知
- DataFrame
  - 无论读取什么数据，都写成DataSet\<Row\> (泛型)
  - 对于SQL语句，只是一条string语句，在编译时不会报**语义错误**
- DataSet
  - 明确声明类型DataSet\<Person\>
  - 若访问不存在的列，在编译时会报错

# KEYS

## DFS

1. DFS中的block与FS中的block的区别
2. DFS中为什么需要备份
3. HDFS中的容错机制

## Hadoop

1. 已经有MPI成熟的并行计算框架，为什么还需要MapReduce？
2. 只有mapper完成后Reducer才可以开始
3. 如何确定数据shuffle到哪个reduce？
4. 如何划分split？
5. shuffle在map端和reduce端的操作
6. 合并（conbine）与归并（merge）的区别
7. reduce输出是否需要[排序](https://hadoop.apache.org/docs/r2.8.0/api/org/apache/hadoop/mapred/Reducer.html)
8. 为什么mapreduce不能实时流计算？
9. **MapReduce执行过程图**
10. 设计理念为计算向数据靠拢

## Spark

1. 线程、进程
   - 除了mapreduce，其余都是多线程实现
   - 线程更轻量化
   - 多核CPU可以同时运行多线程（CPU同时只能运行一个进程）
   - 进程容易debug
2. RDD
   - 运算操作
   - Lineage
3. 宽依赖/窄依赖（有什么用？1.用来划分stage，2.用来容错）

   - 窄依赖
     - 表现为一个父的RDD分区对于于一个子RDD的分区
     - 或多个父的RDD分区对应于一个子的RDD分区
   - 宽依赖
     - **表现为存在一个父RDD的一个分区对应一个子RDD的多个分区**
4. 为什么关心依赖关系？
5. spark体系结构与执行过程图
6. Spark中的线程是什么，进程是什么？
7. spark中job/stage/task对应关系（逻辑执行角度/物理执行角度）
8. spark优化机制
9. spark中的3种容错方式及优缺点
10. 什么时候会将RDD Repartition？
   - 当Join操作为宽依赖，可以经过partition之后变为窄依赖，pipeline执行
11. DataFrame和DataSet的区别
