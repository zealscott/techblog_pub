

在Flink中，有三种迭代类型，一种对应DataStream，两种对应DataSet

### DataStream Iteration

![iteration1](/images/iteration1.png)

![iteration2](/images/iteration2.png)

这里的`iterationBody`接收X的输入，并根据最后的`closeWith`完成一次迭代，另外的数据操作Z退出迭代。

[官方文档参考这里](https://ci.apache.org/projects/flink/flink-docs-stable/dev/datastream_api.html#iterations)

由于DataStream有可能永远不停止，因此不能设置最大的迭代次数，必须显式的指定哪些流需要迭代，哪些流需要输出。

这时候，一般使用`split` transformation 或者  `filter`来实现。

下面定义了一个0-1000的字符流，每次迭代都减1，每个数直到小于等于1时输出（退出循环）：

```java
DataStream<Long> someIntegers = env.generateSequence(0, 1000);

IterativeStream<Long> iteration = someIntegers.iterate();

DataStream<Long> minusOne = iteration.map(new MapFunction<Long, Long>() {
  @Override
  public Long map(Long value) throws Exception {
    return value - 1 ;
  }
});

DataStream<Long> stillGreaterThanZero = minusOne.filter(new FilterFunction<Long>() {
  @Override
  public boolean filter(Long value) throws Exception {
    return (value > 0);
  }
});

iteration.closeWith(stillGreaterThanZero);

DataStream<Long> lessThanZero = minusOne.filter(new FilterFunction<Long>() {
  @Override
  public boolean filter(Long value) throws Exception {
    return (value <= 0);
  }
});
```

对于迭代体`iteration`，每次执行map操作生成`minusOne`，然后将`stillGreaterThanZero`返回给`iteration`，同时将`lessThanZero`输出。

### DataSet Bulk Iteration

![iteration3](/images/iteration3.png)

![iteration4](/images/iteration4.png)

可以发现与DataStream类似，但必须要迭代结束才能有输出。

同时，除了设置最大迭代次数，在`closeWith`中还可以添加第二个DataSet，当其为空时，则退出循环。

**与流计算的区别：**

1. Input会有源源不断的来，且迭代过程中会有数据进入
2. Output随时都可以输出

### DataSet Delta Iteration

由于在图计算中有很多算法在每次迭代中，不是所有TrainData都参与运算，所以定义`Delta`算子。

workset是每次迭代都需要进行更新的，而solution set不一定更新。

![iteration5](/images/iteration5.png)

![iteration6](/images/iteration6.png)

可以通过`iteration`.getWorkset/getSolutionSet得到对应的DataSet，同时分别进行计算，当nextWorkset为空时，退出循环。

在迭代过程中，WorkSet是直接替代，而deltaSet是merge。