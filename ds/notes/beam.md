# SparkV2

## 回顾

### Feature

在第一代的Spark Streaming系统中，其主要特点为：

- 以批处理核心，使用micro-batch模型将流计算转换为批处理
- 流计算和批处理API**可以互用**
  - DStream（特殊的RDD)
  - RDD

### Spark Streaming局限性

Spark streaming难以处理的需求

- Event-time
- Late Data
  - 流数据的三个特征
    - 乱序
    - 延迟
    - 无界
- Session windows
  - 比较难处理，与batch框架相矛盾

## Structured Streaming思路

- 类似Flink，流向表转换
- 流与表的操作统一到DataSet/DataFrameAPI
- **底层引擎依然是批处理**，继续使用micro-batch的模型
  - Continuous query模型还在开发中

## 处理模型

### Unbounded Table

借鉴了Spark中的Dynamic Table实现批流等价转换

### Event time

将Event Time 作为表中的列参与到Window运算中

### Late Data

引入流水线机制

# Beam

Beam系统需要注意什么？

- 同一API
  - 会不会造成严重的性能差异
- 同一编程
  - 低层的两个系统如何实现统一

### WWWH模型

只需要管需要进行说明操作，不关心谁去执行

1. **What** results are calculated?
   - 计算什么结果? (read, map, reduce)
   - 批处理系统可实现
2. **Where** in event time are results calculated? 
   - 在哪儿切分数据? (event time windowing)
   - Windowed Batch
3. **When** in processing time are results materialized?
   - 什么时候计算数据? (triggers)
   - Streaming
4. **How** do refinements of results relate?
   - 如何修正相关的数据?(Accumulation)
   - Streaming + Accumulation

## BeamPipeline

数据处理流水线

- 表示抽象的流程
- 与“Flink流水线机制”不是一个概念