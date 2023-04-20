# Transformation

## 常用函数

1. `map(function)`

   - 将每个元素传递到function中，并返回一个新的数据集

2. `flatmap`

   - 在flatMap中，我们会传入一个函数，该函数对每个输入都会返回一个集合List（而不是一个元素），然后，flatMap把生成的多个集合“拍扁”成为一个集合。

3. `mapToPair()`

   - 返回一个`tuple2`类型的KeyValue对，类型为`JavaPairRDD`

4. `mapValue`

   - 对value进行函数映射

5. `reduceByKey`

   - 按照key对value进行操作

6. `join(RDD)`

   - 按照key进行join，join之后的RDD的value为`tuple2`类型

7. `filter()`

   - 删选出满足函数的元素，并返回一个新数据集

   - 如，筛选出不等于第一行的所有元素：

     - ```java
       filter(line -> !line.equals(fisrtLine))
       ```




### 求平均值

 常见算子之一，一般先将RDD map为（key，1）的形式，再使用ReduceByKey合并，效率较高，更多方法[参考这里](https://blog.csdn.net/gx304419380/article/details/79455833)。

# Action

1. `count()`

   - 返回RDD中元素个数

2. `collect()`

   - 以数组（arrayList）的形式返回数据集中的所有元素，开销很大

3. `first()`

   - 返回数据集中的第一个元素

4. `take(n)`

   - 以数组（ArrayList）的形式返回数据集的前n个元素

5. `reduce()`

   - reduce((a,b)->a+b)，根据两个参数返回一个值，聚合数据集中的元素

   





