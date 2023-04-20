
# 机器学习系统

## 机器学习三要素 

- **模型** 
  - 用**参数**来构造描述客观世界的数学模型 
  - 线性回归模型
  - 对于参数很多的模型，还需要考虑参数容错（LDA）
  - CNN/Tensorflow/Parameter Server -> 适用于从参数入手
- **策略** 
  - 基于已有的**观测数据**进行经验风险最小化 
  - 选择什么参数在代价函数中是最好的？（例如，训练数据的误差平方和）
  - 对于数据之间有关联的算法（PageRank）/GraphLab -> 适用于从数据入手
- **算法** 
  - 优化问题的**计算**求解
  - 适用于Model较为简单，数据没有关联性/Mahout -> 适用于从计算入手
  - 梯度下降
    - 要求必须每次迭代同时更新参数
      - 与BSP模型很像
    - 在某些算法中可以异步更新（系统：GraphLab）

## 机器学习系统设计思路

- 机器学习
  - 强调机器学习方法的精确程度 
  - 评价标准:accuracy/precision/recall 
- 机器学习系统 
  - 强调机器学习过程的运算速度 
  - 评价标准:performance

| 机器学习 | 机器学习系统 |
| -------- | ------------ |
| 模型     | 参数         |
| 策略     | 数据         |
| 算法     | 计算         |

### 思考

- 应该从哪个角度来设计机器学习系统？
  - 考虑模型的复杂度/数据关联性/参数个数
  - 分别从参数/数据/计算入手
- 哪些机器学习过程适合以计算为中心?
  - 线性回归
- 哪些机器学习过程适合以数据为中心？
  - PageRank
- 哪些机器学习过程适合以参数为中心？
  - LDA/CNN

# Mahout

**以计算为中心的机器学习系统**

- MapReduce框架不适合作为机器学习计算
  - 迭代耗时，编程模型很难使用
  - 缺乏声明式
- 使用Spark/Flink作为底层系统
- R/Matlab-likeDSL for declarative implementation of algorithms 
- 自动调优
  - 矩阵平方
    - 分别对行和列进行partition，再分别相乘（Partition），最后相加
    - 对于slim矩阵，可以忽略最后的patition，因为最终的矩阵很小

# GraphLab

**以数据为中心的机器学习系统**

我们之前已经有了基于BSP Model的Pregel，其主要特点是需要进行同步（双屏障），而同步是由最慢的节点决定，造成：

1. 资源的浪费（大部分节点会等待少部分节点收敛）
2. 某些算法可能并不需要同步更新

## 某些机器学习计算的特点

- 异步迭代（Asynchronous Iterative）
  - 参数不一定需要同步更新 
- 动态计算（Dynamic Computation）
  - 某些参数可能快速收敛 
  - 在Pregel中可以通过设置inactive实现
- 可串行化（Serializability） 
  - 计算过程存在一定的顺序依赖关系，或者顺序计算的效果(收敛速度、精确度)更 
  - BSP model中简化了这个问题，但可能影响迭代次数

因此，在GraphLab中，针对这三点分别进行了改进：

- 异步迭代 
  - Pull model 
  - Update function 
- 动态计算 
  - Pull model 
- 可串行化 
  - Scheduler 
  - Consistency Model 

## 计算模型

- 在Giraph中是节点主动发送消息，**获取消息是一个被动的过程，Push model**
- 而在GraphLab中，是主动拉去消息，Pull model
- 异步迭代 
  - Pregel:不支持，**BSP模型是同步迭代** 
  - GraphLab:**通过pull model可以支持异步** 
- 动态计算 
  - Pregel:根据判定条件让某些顶点votetohalt，**但是后续可能还会收到消息**
  - GraphLab:根据判定条件停止计算 updatefunction，**不再pull消息** 
- 调度
  - 与事务处理很像，如何实现？
- Consistency Rules
  - Full Consistency
    - 最强的一致性，但并发度很低
  - Edge Consistency
    - 只对边和中心点进行一致性保证
  - Vertex Consistency
    - 只对顶点进行一致性保证

# Parameter Server

## 设计思路

- 参数与训练数据分开存放 
  - **Server:负责参数** 
  - **Worker:负责训练数据** 
    - 取参数是按需Pull，再将更新后的参数放回去
- 提供同步计算与异步计算模式 
  - 灵活的consistency 
  - 用户选择
- 参数看作key-value pair进行备份 
  - Consistent hashing 

## 计算模式

- Pull/Push/User defined functon
- **提供同步（Sequence）和异步（Eventual）的迭代**
- Trade-off between **Algorithm Efficiency and System Performance** 
  - 计算考虑
    - 异步可能是错的
  - 性能方面
    - 异步更好

## 容错机制

使用一致性哈希和备份的方式

- 为什么不能用zookeeper？
  - 数据量太大
- 前提：这个model是key-value pair，才能够被hash

# KEY

## 机器学习系统

- 机器学习三要素及对应系统
- 应该从哪个角度来设计机器学习系统？
- 哪些算法适合什么系统实现？

## GraphLab

- 与Giraph的区别？（拉去消息/迭代计算/终止条件）
- 如何实现调度？Consistency的方式有哪些？
- GraphLab vs. DB transactions 
  - Consistency类似事务可串行化中的什么概念? 
    - 隔离性
- Pregel vs. GraphLab 
  - BSP模型是哪种consistency? 
    - Full consistency
  - Pregel的vertex-centric编程模型服从vertex consistency model吗? 
    - 服从

## PS

- PS中的参数和数据存在哪里？
- 在PS中，如何得到想要的参数？
- **GraphLab和PS中sequential一样吗?** 
  - GraphLab**强调数据点之间的顺序计算关系** 
  - PS不考察训练数据点之间的关系，**强调多次迭代之间的顺序关系** 
- **GraphLab中的consistency和PS中的 consistency是一样的吗?** 
  - GraphLab中的consistency**解决可串行问题** 
  - PS中的consistency解决**同步/异步计算问题** 
