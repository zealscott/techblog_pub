# 概述

计算机系统中有很多独占资源，依次只能被一个进程使用（打印机等）。

所有的操作系统都具有授权一个进程排他访问某一资源的能力，这个能力包括软件和硬件。

很多进程需要独占的访问几个资源，而这时候，**多个进程可能无限制地等待其他进程将独占资源释放，也就是等待永远不会发生的条件，这种情况叫做死锁。**

## 资源

进程对设备、文件等获得独占性的访问权时有可能会发生死锁，为了尽可能地通用化，将这种需排它使用的对象称为**资源**。

***资源有两类：***

- **可剥夺式资源**可从拥有它的进程处剥夺而没有任何副作用，存储器是一类可剥夺资源。（例如内存）


- **不可剥夺资源**无法在不导致相关计算失败的情况下将其从属主进程处剥夺。（打印机）

# 死锁原理

## 死锁发生的必要条件

**互斥：**任一时刻只允许一个进程使用资源
**请求和保持：**进程在请求其余资源时，不主动释放已经占用的资源
**非剥夺：**进程已经占用的资源，不会被强制剥夺
**环路等待：**环路中的每一条边是进程在请求另一进程已经占有的资源。

## 死锁模型

可以使用图模型来表示，其中**矩形表示资源**（里面的圆点表示实例），**圆圈表示进程**。

没有产生死锁的情况：

![deadlock1](/images/deadlock1.png)

产生死锁：

![deadlock2](/images/deadlock2.png)

可以总结出：

- 如果一个图模型没有环，则不可能有死锁
- 如果一个图存在环
  - 若每一个资源只有一个实例，那么会发生死锁
  - 若每个资源有多个实例，那么可能会发生死锁

# 死锁策略

## 鸵鸟算法

这是最简单的算法。虽然死锁出现以后很棘手，但实践表明，死锁很少发生。

因此，我们可以假装没有这个事情，如果发生了，仅仅需要重启一下就好。

虽然看起来很蠢，但这种方法被包括UNIX等大多数操作系统所使用。

## 死锁的检测和恢复

- 采用这种技术，系统只需监听资源的请求和释放。每次资源被请求或释放时，资源图被更新，同时检测是否存在环路，若存在，则撤销环中的一个进程，直到不产生环路。
- 另外一种更粗略的方法是周期性的检测是否有进程连续阻塞超过一定时间，若发现，则撤销该进程。

## 死锁的预防

按照之前提出的四个必要条件，我们只需要破除其中的一个，则死锁绝对不会发生。

### 破坏互斥

类似于假脱机技术，使用一个守护进程来真正请求资源。

并且，需要把守护进程设计成完整的输出文件就绪后才开始打印。

### 禁止已拥有资源的进程等待其他资源

1. 规定所有进程在开始执行前请求所需要的所有资源
   - 不现实，因为许多进程直到运行才知道它需要多少资源。
   - 这种方案不能最优化资源的使用。
2. 当一个进程请求资源时，先暂时释放当前的所有资源，然后再尝试一次来获得所需的所有资源。

### 破坏不可抢占

这是不太现实和棘手的。

### 消除环路

1. 简单的保证每一个进程在任何时候只占用一个资源
   - 这种限制是不可接受的
2. **给所有资源提供一个全局编号**
   - 进程申请资源必须按照编号顺序
   - 或者仅要求不允许进程请求编号比当前所有占用资源编号低的资源

## 避免死锁

不必对进程强加一些规则来避免死锁，而是通过对每一次资源请求认真的分析来判断是否能安全的分配。

### 银行家算法

#### 基本思想

- 一个小城镇的银行家向一群客户分别承诺了一定的贷款额度，考虑到所有客户不会同时申请最大额度贷款，他只保留较少单位的资金来为客户服务


- 将客户比作进程，贷款比作设备，银行家比作操作系统

#### 算法

- 对每一个请求进行检查，检查如果满足它是否会导致不安全状态。若是，则不满足该请求；否则便满足。


- 检查状态是否安全的方法是看他是否有足够的资源满足某一客户。
- 如果可以，则这笔投资认为是能够收回的，然后检查最接近最大限额的客户，如此反复下去。如果所有投资最终都被收回，则该状态是安全的，最初的请求可以批准。

![deadlock3](/images/deadlock3.png)

其中，a、b是安全的，c是不安全的。

### 资源轨迹图

![deadlock4](/images/deadlock4.png)

### 两阶段加锁法

- 死锁预防的方案过于严格，死锁避免的算法又需要无法得到的信息
- 实际情况采取相应的算法
  - 例如在许多**数据库系统**中，常常需要将若干记录上锁然后进行更新。当有多个进程同时运行时，有可能发生死锁。常用的一种解法是**两阶段加锁法**。
  - 第一阶段，进程试图将其所需的全部记录加锁，一次锁一个记录。
  - 若成功，则开始第二阶段，完成更新数据并释放锁。
  - 若有些记录已被上锁，则它将已上锁的记录解锁并重新开始第一阶段



总结以上策略的优缺点如下：

![deadlock5](/images/deadlock5.png)

# 参考资料

1. Operating System:Design and Implementation,Third Edition 