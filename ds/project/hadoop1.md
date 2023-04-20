# Hadoop操作

## 目录操作

- 在操作之前，需要在hadoop根目录下创建与Linux用户同名的user目录

  ```shell
  ./bin/hdfs dfs -mkdir -p /user/hadoop
  ```

- 之后，**所有的文件都默认放入这个目录下面**，很多命令与Linux命令一致，比如查看当前文件夹：

  ![hadoop21](/images/hadoop21.png)

- 这个`input`是这样创建的：

  ```shell
  ./bin/hfs dfs -mkdir input 
  ```

- 若`/input`，表示在HDFS的根目录创建`input`目录



## 文件操作

### 本地->Hadoop

- 将本地文件移动到hadoop的input文件夹下：

  ![hadoop22](/images/hadoop22.png)

- 查看Hadoop的input文件夹下的文件：

  ![hadoop23](/images/hadoop23.png)

### Hadoop->本地

- 同时，也可以将Hadoop上的文件下载到本地：

  ![hadoop24](/images/hadoop24.png)


### Hadoop之间

- 在Hadoop的文件之间进行传输，与Linux上文件传输无异

- 注意，要使用`-cp`命令，一定要确保目标文件夹存在：

  ![hadoop25](/images/hadoop25.png)

# 配置IDE环境

## 下载IDEA

- 首先在官网下载IDEA到`Download`：

  ![hadoop26](/images/hadooop26.png)

- 然后执行解压命令，解压到`/usr/local`

  ```shell
  sudo tar -xvf ideaIU-2018.2.4.tar.gz -C /usr/local
  ```

- 进入该目录，执行`idea.sh`，进行安装：

  ![hadooop27](/images/hadoop27.png)


## 导入依赖包

1. `/usr/local/hadoop/share/hadoop/common`目录下的：
   - `hadoop-common-xxxx.jar`
   - `hadoop-nfs-xxx.jar`
2. `/usr/local/hadoop/share/hadoop/hdfs/hadoop-hdfs-client.xx.jar`(**3.xx版本需要新加入**)
3. `/usr/local/hadoop/share/hadoop/hdfs`目录下：
   - `hadoop-hdfs-xxx.jar`
   - `hadoop-hdfs-nfs-xxx.jar`
4. `usr/local/hadoop/share/hadoop/common/lib`目录下的所有JAR包
5. `/usr/local/hadoop/share/hadoop/hdfs/lib`目录下的所有JAR包

![hadoop28](/images/hadoop28.png)

## 设置Global Libraries

- 在setting里面设置好Global Libraries后，每次新建工程，若需要这些库（例如Hadoop库），那么需要添加依赖：
  - ![hadoop29](/images/hadoop29.png)
- 可以参考[这里](https://stackoverflow.com/questions/25284800/global-libraries-in-intellij-ide)

# 运行Hadoop文件

## 测试文件

以下文件用于测试HDFS中是否存在一个文件。

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class HDFSFileExist {
    public static void main(String[] args){
        try {
            String fileName = "test";
            Configuration conf = new Configuration();
            conf.set("fs.defaultFS","hdfs://localhost:9000");
            conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
            FileSystem fs = FileSystem.get(conf);
            if(fs.exists(new Path(fileName))){
                System.out.println("file exist!");
            }else{
                System.out.println("file not exist");
            }


        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

这里，需要检测的文件名称问`test`，没有给出路径全称，则表示采用了相对路径，就是当前登录Linux系统的用户`Hadoop`，在对应的目录下是否存在`test`，也就是`/usr/hadoop`目录下是否存在test文件。

## 结果

在`IDEA`中直接运行，可得到如下结果：

![hadoop210](/images/hadoop210.png)

## 将项目打包成JAR包

- 在`IDEA`中，右键项目，选择`open module setting`，进入`Artifact`，点`+`号：

  ![hadoop211](/images/hadoop211.png)

- 选择`with dependencied`，新建一个：

  ![hadoop212](/images/hadoop212.png)

- 选择`Main class`，其余默认

- 然后build artifact：

  ![hadoop213](/images/hadoop213.png)

- build之后，在对应文件夹中找到打包的JAR包：

  ![hadoop214](/images/hadoop214.png)

- 在本地尝试是否打包成功：

  ![hadoop215](/images/hadoop215.png)

  出现如图结果就表示成功！