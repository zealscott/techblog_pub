
# 集群上使用

## jar包

- 首先将之前`FileExist`文件进行打包，得到`.jar`文件：
  - ![hadoop231](/images/hadoop231.png)
- 将其拷贝到集群中，并使用`hadoop jar`命令运行：
  - ![hadoop233](/images/hadoop233.png)

# WordCount

## 添加依赖

- 首先我们需要新建一个`WordCount`项目，首先要添加`Hadoop`的包依赖
  - `/usr/local/hadoop/share/hadoop/common`
    - `hadoop-common-xxx.jar`
    - `hadoop-nfs-xxx.jar`
  - `/usr/local/hadoop/share/hadoop/common/lib` 下的所有Jar包
  - `/usr/local/hadoop/share/hadoop/mapreduce`该目录下所有JAR包
  - `/usr/local/hadoop/share/hadoop/mapreduce/lib`目录下所有JAR包
  - ![hadoop232](/images/hadoop232.png)

## 编写程序

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class WordCount {
    public WordCount () {
    }

    public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        public TokenizerMapper () {
        }
        public void map(Object key, Text value, Mapper<Object, Text, Text, IntWritable>.Context context
        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                this.word.set(itr.nextToken());
                context.write(this.word, one);
            }
        }
    }

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

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = (new GenericOptionsParser(conf, args)).getRemainingArgs();
        if (otherArgs.length < 2) {
           System.err.println("Usage: wordcount <in>[<in>...] <out>");
           System.exit(2);
        }
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(WordCount.TokenizerMapper.class);
        job.setCombinerClass(WordCount.IntSumReducer.class);
        job.setReducerClass(WordCount.IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        for (int i = 0; i < otherArgs.length-1; i++) {
            FileInputFormat.addInputPath(job, new Path(otherArgs[i]));
        }
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[otherArgs.length-1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

## 打包成JAR包

- 打开Project Structure：
  - ![hadoop234](/images/hadoop234.png)
- 进行编译：
  - ![hadoop235](/images/hadoop235.png)
- 生成并查看JAR包：
  - ![hadoop236](/images/hadoop236.png)

## 本地伪分布式运行

- 创建两个文件作为输入，内容为：

  - >I love Spark
    >I love Hadoop
    >
    >Hadoop is good
    >Spark is fast

- 将本地文件放入`hdfs`中：

  - ```shell
    hdfs dfs -mkdir -p /user/hadoop/input
    hdfs dfs -put ./wordfile1.txt input
    hdfs dfs -put ./wordfile2.txt input
    ```

- 在`hdfs`中查看：

  - ```shell
    hdfs dfs -ls input
    ```

  - ![hadoop237](/images/hadoop237.png)

- 运行：

  - ```shell
    hadoop jar WordCount.jar input output
    ```

- 查看结果：

  - ```shell
    hdfs dfs -cat output/*
    ```

  - ![hadoop238](/images/hadoop238.png)


# 集群上运行

- 首先将JAR包和文件放入集群：

  - ![hadoop239](/images/0060lm7Tly1fw5guehs03j30au01emx0.png)

- 将其拷贝到`HDFS`中：

  - ```shell
    hdfs dfs -mkdir -p /user/hadoop7/input
    hdfs dfs -put ./wordfile1.txt input
    hdfs dfs -put ./wordfile2.txt input
    ```

- 查看文件：

  - ![hadoop2310](/images/hadoop2310.png)

- 运行：

  - ```shell
    hadoop jar WordCount.jar input output
    ```

  - ![hadoop2311](/images/hadoop2311.png)

  - 运行成功

    - ![hadoop2312](/images/hadoop2312.png)

- 查看生成文件

  - ```shell
    hdfs dfs -cat /user/hadoop7/output/*
    ```

  - ![hadoop2313](/images/hadoop2313.png)

- 查看集群运行情况

  - 在连接VPN时，在浏览器中输入`10.11.6.91:50070`
  - ![hadoop2314](/images/hadoop2314.png)

- 