
# 基本类型

## Text

主要接口和方法可以[参考这里](https://hadoop.apache.org/docs/r2.6.2/api/org/apache/hadoop/io/Text.html)。

### 构造

- ```java
  private Text word = new Text()
  ```

- 这里的`Text`类与`String`类似

### 方法

- `set(String string)`
  - 复制一个`String`类型到当前`Text`中

## IntWritable

### 构造

- ```java
  private IntWritable one = new IntWritable(1)
  ```


### 方法

- `get()`
  - 得到这个值
- `set(int)`
  - 设置值




# 文件操作

## HDFS文件

得到一个HDFS文件系统，这样就可以使用常规的文件操作方式进行操作

```java
Configuration conf = new Configuration();
FileSystem hdfs = FileSystem.get(conf);
```

## 文件CRUD

创建或追加文件并写入

```java
outputStream = fs.create(dstPath)；
outputStream = fs.append(dstPath);
outputStream.write(contents.getBytes("utf-8"));
```

删除文件（夹）

```java
hdfs.delete(outputDir, true); // recursive delete
```

移动文件（重命名）

```java
hdfs.rename(actualPath, cachePath);
```

# MapReduce

## InputFormat

1. `TextInputFormat`

   - `TextInputFormat`是默认的`InputFormat`，每条记录是一行输入
   - Key是`LongWriteable`类型 ，存储该行在整个文件中的字节偏移量
     - 一般来说，很难讲字节偏移量转为行号，除非每一行的偏移量都相同
   - Value是这行的内容，不包含任何行终止符（回车符和换行符），被打包成一个`Text`对象

2. `KeyValueTextInputFormat`

   - 一般来说，偏移量没有太大用，通常文件时暗转键值对组织的，使用某个分界符进行分割。

   - Hadoop的默认分隔符是制表符，可以这样修改：

   - ```java
     // 创建配置信息
     Configuration conf = new Configuration();
     // 设置行的分隔符，这里是制表符，第一个制表符前面的是Key，第一个制表符后面的内容都是value
     conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, "\t");
     // 设置输入数据格式化的类
     job.setInputFormatClass(KeyValueTextInputFormat.class);
     ```

3. `NLineInputFormat`

   - 通过`TextInputFormat`和`KeyValueTextInputFormat`，每个mapper受到的输入行数不同，取决于分片的大小和行的长度。
   - 如果希望每个mapper受到的行数相同，需要将`NLineInputFormat`作为`InputFormat`。
   - 与`TextInputFormat`一样，键是文件中的字节偏移量，值是行本身。

## Mapper

```java
public class TokenCounterMapper 
   extends Mapper<Object, Text, Text, IntWritable>{
    
   private final static IntWritable one = new IntWritable(1);
   private Text word = new Text();
   
   public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
     StringTokenizer itr = new StringTokenizer(value.toString());
     while (itr.hasMoreTokens()) {
       word.set(itr.nextToken());
       context.write(word, one);
     }
   }
 }
```

- 这里以`WordCount`为例，首先，`TokenCounterMapper`继承了`Mapper`泛型类，并确定`keyIn, valueIn,keyOut,valueOut`的类型。

- 然后定义需要的变量

- 同时重写`map`函数，其中的参数的类型要与之前的类的类型一致，注意，`context`类是用来传递数据以及其他运行状态信息，这里`Context context`自动匹配到`Text, IntWritable`，也可以这样写：

  - ```java
    public void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context)
    ```

  - 这里直接指定本Mapper类中的`Context`类型。

- 如论如何，程序最终都需要向`Context.write()`中传递两个参数，也就是map的`keyOut,valueOut`。

- 注意，当mapper段的输出kv与最后reducer的kv数据类型不一致时，需要在主程序中指明输出类型：`setMapOutputKeyClass`，`setMapOutputValueClass`。


### Setup

在`mapper`端可以使用`setup`函数，该函数在进入map阶段前运行，每个job只会运行一次：

```java
public void setup(Context context) throws IOException, InterruptedException {}
```




## Context

- 在Mapper中的map、以及Reducer中的reduce都有一个Context的类型。

- Context应该是用来传递数据以及其他运行状态信息，map中的key、value写入context，让它传递给Reducer进行reduce，而reduce进行处理之后数据继续写入context，继续交给Hadoop写入hdfs系统。

- 理解：

  - [mapreduce中的context类](https://blog.csdn.net/SONGCHUNHONG/article/details/50435717?utm_source=blogxgwz4)
  - The Context object allows the Mapper/Reducer to interact with the rest of the Hadoop system.


## Configuration

- `Configuration(boolean loadDefaults)`
- Configuration可以用来在MapReduce任务之间共享信息，当然这样共享的信息是在作业中配置，一旦作业中的map或者reduce任务启动了，configuration对象就完全独立。所以共享信息是在作业中设置的。
- 当`loadDefaults`为false时，Configuration对象就不会将通过addDefaultResource(String resource)加载的配置文件载入内存。但是会将通过addResource(...)加载的配置文件载入内存。
- [参考这里](https://www.cnblogs.com/wolfblogs/p/4156403.html)

## Reduce

```java
public static class IntSumReducer
     extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                           Reducer<Text,IntWritable,Text,IntWritable>.Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            this.result.set(sum);
            context.write(key, this.result);
        }
    }
```

- 同样以`WordCount`为例，这里与map很相似，`IntSumReducer`继承了`Reducer`泛型类，并确定`keyIn, valueIn,keyOut,valueOut`的类型。
- 注意，在定义`reduce`函数时，这里的输入为`<key, value-list>`形式，因此需要`Iterable`类型作为`valueIn`。
- 最后与`map`一样，写入`context`中。

可以设置Reducer的个数：

```java
job.setNumReduceTasks(6);
```

## Main

```java
public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();		// 程序运行时参数	
        String[] otherArgs = (new GenericOptionsParser(conf, args)).getRemainingArgs();
        if (otherArgs.length < 2) {
           System.err.println("Usage: wordcount <in>[<in>...] <out>");
           System.exit(2);
        }
        Job job = Job.getInstance(conf, "word count");		// 设置环境参数
        job.setJarByClass(WordCount.class);					// 设置整个程序的类名
    
        job.setMapperClass(WordCount.TokenizerMapper.class);   // 添加Mapper类
        job.setCombinerClass(WordCount.IntSumReducer.class);   // 添加Combiner类
        job.setReducerClass(WordCount.IntSumReducer.class);   // 添加Reducer类
    
    	job.setMapOutputValueClass(IntWritable.class);		// 设置mapper输出类型
        job.setOutputKeyClass(Text.class);					// 设置mapper输出类型
        job.setOutputKeyClass(Text.class);					// 设置reducer输出类型
        job.setOutputValueClass(IntWritable.class);			// 设置reducer输出类型
        for (int i = 0; i < otherArgs.length-1; i++) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));   // 设置输入文件
        }
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[otherArgs.length-1]));   // 设置输出文件
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
```

### 设置

- `conf.set("mapreduce.output.textoutputformat.separator", " ");`
  - 可以更改输出的kv分隔符，默认为`\t`


### distributed cache

- 可设置`Distribution cache`

  - ```java
    job.addCacheFile(cacheMeansPath.toUri());
    ```

  - 使用时：

  - ```java
    String localCacheFiles = context.getLocalCacheFiles()[0].getName();
    BufferedReader br = new BufferedReader(new FileReader(localCacheFiles));
    ```


## 组合式MapReduce

### 隐式依赖描述（不建议）

```java
private void execute() throws Exception{
    String tempOutPutPath = OutPutPath + "_tmp";
    runWordCountJob(inputPath,tempOutPutPath);
    runFrequenciesJob(tempOutPutPath,OutPutPath);    
}

private int runWordCountJob(String inputPath, String tempOutPutPath) throws Exception{
     Configuration conf = new Configuration();		// 程序运行时参数	
    
    ....
}

private int runFrequenciesJob(String tempOutPutPath, String OutPutPath) throws Exception{
     Configuration conf = new Configuration();		// 程序运行时参数	
    
    ....
}
```

### 显示依赖描述

```java
Configuration job1conf = new Configuration();
Job job1 = new Job(job1conf,"Job1");
.........//job1 其他设置
Configuration job2conf = new Configuration();
Job job2 = new Job(job2conf,"Job2");
.........//job2 其他设置
Configuration job3conf = new Configuration();
Job job3 = new Job(job3conf,"Job3");
.........//job3 其他设置
    
job3.addDepending(job1);//设置job3和job1的依赖关系
job3.addDepending(job2);

JobControl JC = new JobControl("123");
JC.addJob(job1);//把三个job加入到jobcontorl中
JC.addJob(job2);
JC.addJob(job3);
JC.run();
```

## 链式MapReduce

```java
public void function throws IOException {
        Configuration conf = new Configuration();
        Job job = new Job(conf);
        job.setJobName("ChianJOb");
        // 在ChainMapper里面添加Map1
        Configuration map1conf = new Configuration(false);
        ChainMapper.addMapper(job, Map1.class, LongWritable.class, Text.class,
                Text.class, Text.class, true, map1conf);
        // 在ChainReduce中加入Reducer，Map2；
        Configuration reduceConf = new Configuration(false);
        ChainReducer.setReducer(job, Reduce.class, LongWritable.class,
                Text.class, Text.class, Text.class, true, map1conf);
        Configuration map2Conf = new Configuration();
        ChainReducer.addMapper(job, Map2.class, LongWritable.class, Text.class,
                Text.class, Text.class, true, map1conf);
        job.waitForCompletion(true);
    }
```

