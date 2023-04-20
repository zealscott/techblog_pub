+++
title = "使用 Docker 配置 hadoop/spark"
slug = "docker1"
tags = ["distributed system","project"]
date = "2018-11-03T13:12:38+08:00"
description = ""

+++


# 分别安装hadoop和spark镜像

## 安装hadoop镜像

选择的docker[镜像地址](https://hub.docker.com/r/uhopper/hadoop/)，这个镜像提供的hadoop版本比较新，且安装的是jdk8，可以支持安装最新版本的spark。

```shell
docker pull uhopper/hadoop:2.8.1
```

## 安装spark镜像

如果对spark版本要求不是很高，可以直接拉取别人的镜像，若要求新版本，则需要对dockerfile进行配置。

### 环境准备

1. 下载sequenceiq/spark镜像构建源码

   ```shell
   git clone https://github.com/sequenceiq/docker-spark
   ```

2. 从Spark官网下载Spark 2.3.2安装包

   - 下载地址：http://spark.apache.org/downloads.html

3. 将下载的文件需要放到docker-spark目录下

4. 查看本地image，确保已经安装了hadoop

5. 进入docker-spark目录，确认所有用于镜像构建的文件已经准备好

   - ![docker11](/images/docker11.png)

### 修改配置文件

- 修改Dockerfile为以下内容

```makefile
    FROM sequenceiq/hadoop-docker:2.7.0
    MAINTAINER scottdyt
    
    #support for Hadoop 2.7.0
    #RUN curl -s http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.6.tgz | tar -xz -C /usr/local/
    ADD spark-2.3.2-bin-hadoop2.7.tgz /usr/local/
    RUN cd /usr/local && ln -s spark-2.3.2-bin-hadoop2.7 spark
    ENV SPARK_HOME /usr/local/spark
    RUN mkdir $SPARK_HOME/yarn-remote-client
    ADD yarn-remote-client $SPARK_HOME/yarn-remote-client
    
    RUN $BOOTSTRAP && $HADOOP_PREFIX/bin/hadoop dfsadmin -safemode leave && $HADOOP_PREFIX/bin/hdfs dfs -put $SPARK_HOME-2.3.2-bin-hadoop2.7/jars /spark && $HADOOP_PREFIX/bin/hdfs dfs -put $SPARK_HOME-2.3.2-bin-hadoop2.7/examples/jars /spark 
    
    
    ENV YARN_CONF_DIR $HADOOP_PREFIX/etc/hadoop
    ENV PATH $PATH:$SPARK_HOME/bin:$HADOOP_PREFIX/bin
    # update boot script
    COPY bootstrap.sh /etc/bootstrap.sh
    RUN chown root.root /etc/bootstrap.sh
    RUN chmod 700 /etc/bootstrap.sh
    
    #install R 
    RUN rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    RUN yum -y install R
    
    ENTRYPOINT ["/etc/bootstrap.sh"]
 ```

- 修改bootstrap.sh为以下内容

```shell
  #!/bin/bash

  : ${HADOOP_PREFIX:=/usr/local/hadoop}

  $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh

  rm /tmp/*.pid

  # installing libraries if any - (resource urls added comma separated to the ACP system variable)
  cd $HADOOP_PREFIX/share/hadoop/common ; for cp in ${ACP//,/ }; do  echo == $cp; curl -LO $cp ; done; cd -

  # altering the core-site configuration
  sed s/HOSTNAME/$HOSTNAME/ /usr/local/hadoop/etc/hadoop/core-site.xml.template > /usr/local/hadoop/etc/hadoop/core-site.xml

  # setting spark defaults
  echo spark.yarn.jar hdfs:///spark/* > $SPARK_HOME/conf/spark-defaults.conf
  cp $SPARK_HOME/conf/metrics.properties.template $SPARK_HOME/conf/metrics.properties

  service sshd start
  $HADOOP_PREFIX/sbin/start-dfs.sh
  $HADOOP_PREFIX/sbin/start-yarn.sh
  CMD=${1:-"exit 0"}
  if [[ "$CMD" == "-d" ]];
  then
    service sshd stop
    /usr/sbin/sshd -D -d
  else
    /bin/bash -c "$*"
  fi
```

### 构建镜像


  ```shell
  		docker build --rm -t scottdyt/spark:2.3.2 .
  ```

  ![docker12](/images/docker12.png)

  ### 查看镜像

  ![docker13](/images/docker13.png)



启动一个spark2.3.1容器

```shell
	docker run -it -p 8088:8088 -p 8042:8042 -p 4040:4040 -h sandbox scottdyt/spark:2.3.2 bash
```

启动成功：

![docker14](/images/docker14.png)

# 安装spark-hadoop镜像

如果想偷懒一点，直接安装装好spark和hadoop的镜像，镜像地址在[这里](https://hub.docker.com/r/uhopper/hadoop-spark/)。

或者直接在终端输入：

```shell
docker pull uhopper/hadoop-spark:2.1.2_2.8.1
```

安装完成：

![docker15](/images/docker15.png)

## 参考

1. https://blog.csdn.net/farawayzheng_necas/article/details/54341036

