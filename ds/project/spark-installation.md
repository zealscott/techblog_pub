
## 安装Spark

首先在官网上安装对应版本，因为已经安装了`hadoop`，选择`without hadoop`版本。

执行解压、修改文件名、配置文件等操作：

```
sudo tar -zxf spark-2.3.2-bin-without-hadoop.tgz -C /usr/local
 cd /usr/local
sudo mv ./spark-2.3.2-bin-without-hadoop/ ./spark
sudo chown -R hadoop:hadoop ./spark
cd spark/
cp ./conf/spark-env.sh.template  ./conf/spark-env.sh


export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)

vim conf/spark-env.sh

```

同时，将`/usr/local/spark/bin`目录加入系统PATH：`~/.bashrc`，并刷新`source ~/.bashrc`。

## Spark Shell

执行`spark shell`：

```
bin/run-example SparkPi
bin/spark-shell
```

出现如下界面：

![spark21](/images/spark21.png)

## 测试Spark shell

![spark22](/images/spark22.png)

## 浏览器查看

启动Spark shell时后，在浏览器中输入`localhost:4040`：

![spark23](/images/spark23.png)

## 文件访问

首先访问本地的文件：

```scala
val textFile = sc.textFile("file:///home/hadoop/Documents/distribution/Kmeans2/README.txt")
textFile.first()
```

![spark24](/images/spark24.png)

访问HDFS上的文件

```scala
val textFile = sc.textFile("hdfs://localhost:9000/user/hadoop/input/k-means.dat")
textFile.first()
```

![spark25](/images/spark25.png)

在这里也可以不指定localhost，以下三种方式都是等价的：

```scala
val textFile = sc.textFile("hdfs://localhost:9000/user/hadoop/input/k-means.dat")
val textFile = sc.textFile("/user/hadoop/input/k-means.dat")
val textFile = sc.textFile("input/k-means.dat")
```

## WordCount

```scala
val textFile = sc.textFile("file:///usr/local/spark/README.md")
val wordCount = textFile.flatMap(line=>line.split("")).map(word=>(word,1)).reduceByKey((a,b)=>a+b)
wordCount.collect()
```

![spark26](/images/spark26.png)

- 在这里，`textFile`包含多行文本内容，`flatMap`会遍历其中的每行文本内容，当遍历当一行文本内容时，会把本行内容赋值给变量`line`，并执行`Lambda`表达式`line => line.split("")`。
- 这里采用每个单词分隔符，切分得到拍扁的单词集合。
- 然后执行`map`函数，遍历这个集合中的每个单词，将输入的`word`构建得到一个映射（`Map`是一种数据结构），这个映射`key`是word，`value`为1.
- 得到映射后，包含很多`(key，value)`，执行reduce，按照key进行分布，然后使用给定的函数对相同`key`的多个`value`进行聚合操作，得到聚合后的`(key，value)`

## 在集群中访问

```scala
spark-shell --master spark://10.11.6.91:7077
val textFile = sc.textFile("hdfs://10.11.6.91:9000/README.md")
textFile.count()
```

