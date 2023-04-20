# 单个MapReduce

## 单元运算

- 以`WordCount`为例
- 分别编写Map和Reduce函数
  - ![mr1](/images/mr1.png)
  - ![mr2](/images/mr2.png)
- 编写`main`方法，设置环境变量，进行注册：
  - ![mr3](/images/mr3.png)

## 二元编程

## Join

- 对于 `input`，来自不同的关系表，对于`MapReduce`而言都是文件
- 在`Map`过程中，**需要标记来自哪个关系表**
  - 把来自 R的每个元组 `<a,b >`转换成一个键值对 `<b, <R,a >>` ，其中的键就是属性 B的值
  - 把来自 S的每个元组 `<b,c >`，转换成一个键值对`<b,<S,c>>`
- `Reduce` 过程
  - 具有相同 B值的元组被发送到同一个 `Reduce `中
  - 来自 关系 R和S的、具有相同属性 B值的元组进行合并
  - 输出则是连接后的元组 `<a,b,c >`，通常写到一个单独的输出文件中

对于二元运算，例如`Join`、`交集`、`并集`都差不多，首先需要标记来自哪个关系表，然后再处理。

# 组合式MapReduce

- 将任务划分为若干子任务，各任务之间存在依赖关系
- 多次`Join`也可以认为是组合式的任务
  - ![mr4](/images/mr4.png)

### 程序实现

#### 隐式依赖描述

- 如何表示`Job`之间有依赖关系
  - 自己编程实现：
  - ![mr5](/images/mr5.png)

#### 显式依赖描述

- 好处：
  - 系统能拿到调度信息，避免上个程序运行失败导致后面出错
  - 如果自己编程，例如`J4/J5`都依赖于`J3`，其中`J4/J5`一定会有一个顺序，而如果让系统调度，可以利用调度策略效率最大化（通常短作业优先）
- 在config中实现：
  - ![mr6](/images/mr6.png)

# 链式MapReduce

- 例子：词频统计后，过滤掉词频高于10的
- `WordCount`程序已经写好，不能修改
- `Map`可以串很多`ChainMapper`，`Reducer`也可以串很多`ChainReducer`
  - 注意，这里的`ChainReducer`为`Mapper`
  - ![mr7](/images/mr7.png)
  - 

### 规则

- 整个`Job`只有一个`Reduce`
  - 整个框架只允许一次`Shuffle`
  - 进行`Map`不会造成数据重新排列，不会改变`MapReduce`整体框架

### 编程实现

![mr8](/images/mr8.png)

# 迭代MapReduce

- 许多机器学习算法都需要进行迭代（牛顿迭代、EM算法）
- **迭代式任务的特征：**
  - 整个任务一系列子的循环构成
  - 子任务的执行操作是完全相同的
  - 一个子任务的输出是下一个子任务的输入
  - 一个子任务是一个MapReduce Job
- 迭代多少次，就相当于运行多少次`MapReduce`
- 迭代`MapReduce`示意
  - 每一迭代结束时才将结果写入`HDFS`，下一步将结果读出
  - 非常浪费资源和IO

### 编程

- `runlteration()`实现一个`MapReduce Job`
  - ![mr9](/images/mr9.png)
- 判断条件为满足阈值或者迭代次数
  - 有时候并不关心具体的精确数值，只关心偏序关系（`PageRank`）

# Distribute Cache

- 当表的大小差异很大时，使用`Join`会导致大量的数据移动：
  - 编程时将`小表`广播出去（每个节点上发一份，移动计算）
  - ![mr10](/images/mr10.png)
  - ![mr11](/images/mr11.png)
  - 例如，在`Kmeans`中，可以将`中心点`广播出去

## 编程实现

- 声明

  - ```java
    Job job= new Job(); 
    job.addCacheFile (new Path(filename).toUri ());
    ```

- 使用

  - ```java
    Path[] localPaths = context.getLocalCacheFiles();
    ```

# Hadoop Streaming

- `Hadoop`基于Java开发，但`MapReduce`编程不仅限于`Java`语言

- 提供一个编程工具，可以允许用户使用任何可执行文件
  
- 但可能会有bug
  
- 多种语言混合编程
  
- [C/C++ 与 python通信](https://www.zhihu.com/question/23003213)
  
- 原理

  - ![mr12](/images/mr12.png)
