#  运行环境

## Spark安装与部署（Standalone）

### 解压缩文件

将 `spark 压缩包` 上传至`Linux`并解压，放置在指定路径。
```linux
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module 
cd /opt/module  
mv spark-3.0.0-bin-hadoop3.2 spark-local 
```
### 修改配置文件

1. 配置文件位于 `/opt/apps/spark/conf`目录

   将`spark-env.sh.template`重命名为`spark-env.sh`。添加如下内容：

   ```
   export SCALA_HOME=/usr/local/bigdata/scala
   export JAVA_HOME=/usr/local/bigdata/java/jdk1.8.0_211
   export HADOOP_HOME=/usr/local/bigdata/hadoop-2.7.1
   export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
   SPARK_MASTER_IP=Master
   SPARK_LOCAL_DIRS=/usr/local/bigdata/spark-2.4.3
   SPARK_DRIVER_MEMORY=512M
   ```

2. 将`slaves.template`重命名为`slaves`
   修改为如下内容：

   ```
   Slave01
   Slave02
   ```

### 配置环境变量

1. 使用如下命令修改环境变量

   `vi /etc/profile`

2. 进入后添加一下内容后保存推出

    ```linux
    export SPARK_HOME=/opt/apps/spark-loca
    export PATH=$SPARK_HOME/bin:$PATH
    ```
    
3. 重新加载配置文件
    ```
    source /etc/profile
    ```
    
    
    
    

### 启动Spark

先启动`hadoop`

```
cd $HADOOP_HOME/sbin/
./start-all.sh
```


再启动Spark
```
cd $SPARK_HOME/sbin/
./start-allsh
```



<font>注意</font> 由于我们已经配置环境变量，以执行`start-dfs.sh`和`start-yarn.sh`可以不切换到当前目录下，但是`start-all.sh`、`stop-all.sh`和`/start-history-server.sh`这几个命令`hadoop`目录下和`spark`目录下都同时存在，所以为了避免错误，最好切换到绝对路径下。

测试：http://192.168.52.20:8080/ 

### 配置Scala环境

安装：

```
tar zxf scala-2.12.5.tgz -C /opt/apps/
```

配置环境变量：

```
export SCALA_HOME=/opt/apps//scala-2.12.5
export PATH=/usr/local/bigdata/scala-2.12.5/bin:$PATH
```

测试：

```
scala -version #命令

# 	Scala code runner version 2.12.5 -- Copyright 2002-2018, LAMP/EPFL  and Lightbe
```

#### 启动Spark-shell

执行`spark-shell --master spark://master:7077`命令，启动spark shell。

```
hadoop@Master:~$ spark-shell --master spark://master:7077
19/06/08 08:01:49 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://Master:4040
Spark context available as 'sc' (master = spark://master:7077, app id = app-20190608080221-0002).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/

Using Scala version 2.11.12 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_211)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

## Spark安装与部署（Spark on Yarn）
### 解压缩文件

将 `spark 压缩包` 上传至`Linux`并解压，放置在指定路径。
```linux
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module 
cd /opt/module  
mv spark-3.0.0-bin-hadoop3.2 spark-local 
```


### 修改配置文件

1. 配置文件位于 `/opt/apps/spark/conf`目录

   将`spark-env.sh.template`重命名为`spark-env.sh`。添加如下内容：

   ```
   # JDK安装路径
   export JAVA_HOME=/opt/apps/jdk
   YARN_CONF_DIR=/opt/apps/hadoop/etc/hadoop
   # WebUi访问的端口号为18080
   # 指定历史服务器存储路径（需提前创建
   export SPARK_HISTORY_OPTS="
   -Dspark.history.ui.port=18080
   -Dspark.history.fs.logDirectory=hdfs://Snode1:8020/directory
   -Dspark.history.retainedApplications=30"
   ```
   
2. 将`slaves.template`重命名为`slaves`
   改为如下内容：
   
    ```
    Slave01
    Slave02
    ```
   
3. 打开`spark-defaults.conf `    添加
	```linux
	# 开启日志
	spark.eventLog.enabled true
	# 存储路径
	spark.eventLog.dir hdfs://Snode1:8020/directory
	# 和yarn作一个关联
	spark.yarn.historyServer.address=Snode1:18080
	spark.history.ui.port=18080
	```
	
4. `yarn`配置

  ```
  <configuration>
  
  <!-- Site specific YARN configuration properties -->
      <!-- 指定yarn的shuffle技术-->
      <property>
          <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
      </property>
      <!-- 指定resourcemanager的主机名-->
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>Snode1</value>
      </property>
          <!-- 判断是否启动一个线程检查每个任务正使用的物理内存量 -->
          <property>
              <name>yarn.nodemanager.pmem-check-enabled</name>
              <value>false</value>
          </property>
          <!-- 判断是否启动一个线程检查每个任务正使用的虚拟内存量 -->
          <property>
              <name>yarn.nodemanager.vmem-check-enabled</name>
              <value>false</value>
          </property>
  </configuration>
  ```

   

### 配置环境变量

1. 使用如下命令修改环境变量

   `vi /etc/profile`

2. 进入后添加一下内容后保存推出

    ```linux
    export SPARK_HOME=/opt/apps/spark-loca
    export PATH=$SPARK_HOME/bin:$PATH
    ```
    
3. 重新加载配置文件
    ```
    source /etc/profile
    ```
    
### 运行spark官方例子

1. 启动 hdfs ,yarn 

   + `start-dfs.sh`

   + `start-yarn.sh`

2. 测试yarn提交spark任务

    ```linux
       spark-submit --master yarn --class org.apache.spark.examples.SparkPi /opt/apps/spark/examples/jars/spark-examples_2.12-3.0.0.jar 104
    ```

3.  通过web界面查看运状态
   + ![image-20211206092126033](C:\Users\郭济\AppData\Roaming\Typora\typora-user-images\image-20211206092126033.png)

## Kafka 安装与配置

### 2. ZooKeeper安装与配置

+ 安装解压``zookeeper`

+ 修改环境变量

进入`zookeeper/conf`

```
1. mv zoo.*.cfg zoo.cfg
2. vi zoo.cfg


dataDir=/tmp/zookeeper/data
dataLogDir=/tmp/zookeeper/log

server.1=192.168.52.20:2888:3888
server.2=192.168.52.21:2888:3888
server.3=192.168.52.22:2888:3888

在/tmp/zookeeper/data目录下写入1到myid

scp 发送到其它集群
并修改 myid 其他集群myid 不能相同 

```

### 配置kafka

##### 2.2.3.4 **配置broker(server.properties)**

```shell
[root@hadoop01 kafka_2.11-1.1.1]# vi ./config/server.properties
```

该配置文件修改如下内容：

```shell
broker.id=1

log.dirs=/opt/apps/kafka/log/kafka-logs-1

zookeeper.connect=Snode1:2181,Snode2:2181,Snode3:2181/kafka

listeners=PLAINTEXT://Snode1:9092

advertised.listeners=PLAINTEXT://Snode1:9092
```

##### 2.2.3.5 **配置生产者(producer.properties)**

```
[root@hadoop01 kafka_2.11-1.1.1]# vi ./config/producer.properties
```

修改 如下内容：

```shell
# list of brokers used for bootstrapping knowledge about the rest of the cluster
# format: host1:port1,host2:port2 ...
bootstrap.servers=Snode1:9092,Snode2:9092,Snode3:9092
```

##### 2.2.3.6 **配置消费者(consumer.properties)**

```
[root@hadoop01 kafka_2.11-1.1.1]# vi ./config/consumer.properties
```

修改内容如下：

```shell
# list of brokers used for bootstrapping knowledge about the rest of the cluster
# format: host1:port1,host2:port2 ...
bootstrap.servers=Snode1:9092,Snode2:9092,Snode3:9092

# consumer group id
group.id=group-test
```

##### 2.2.3.7 **分发**

分发已经配置好的kafka目录到hadoop02和hadoop03节点的/usr/local目录下。

```shell
[root@hadoop01 kafka_2.11-1.1.1]# scp -r /opt/apps/kafka/ Snode1:/opt/apps/
[root@hadoop01 kafka_2.11-1.1.1]#scp -r /opt/apps/kafka/ Snode2:/opt/apps/
```

修改hadoop02节点上server.properties中的id和host:

```shell
[root@hadoop02 kafka_2.11-1.1.1]# vi ./config/server.properties
修改如下内容：
broker.id=2   ##修改
log.dirs=/opt/apps/kafka/log/kafka-logs-2   ##修改
listeners=PLAINTEXT://Snode2:9092		##修改
advertised.listeners=PLAINTEXT://Snode2:9092  ##修改
```

修改hadoop03节点上server.properties中的id和host:

```shell
[root@hadoop03 kafka_2.11-1.1.1]# vi ./config/server.properties
修改如下内容：
broker.id=3   ##修改
log.dirs=/opt/apps/kafka_2.12/log/kafka-logs-3   ##修改
listeners=PLAINTEXT://Snode3:9092		##修改
advertised.listeners=PLAINTEXT://Snode3:9092  ##修改
```

到此为止，kafka的集群配置完成。

### 启动kafka

依次启动

```
# Snode1
zkServer.sh start
# Snode2
zkServer.sh start
# Snode3
zkServer.sh start

# 查看zookeeper状态
# Snode1
zkServer.sh status

# 依次再每个节点启动kafka的broker
nohup ./bin/kafka-server-start.sh ./config/server.properties > /var/log/kafka.log 2>&1 &

# 前台启动：
bin/kafka-server-start.sh config/server.properties

# jps 测试

kafka生产者命令
kafka-console-producer.sh --broker-list Snode1:9092 --topic test

kafka消费者命令
kafka-console-consumer.sh --bootstrap-server Snode1:9092--topic test --from-beginning

关闭kafka
kafka-server-stop.sh Snode1.properties &

Kafka TopicName 查看

kafka-topics.sh --list --zookeeper Snode1:2181

Kafka TopicName 删除

kafka-topics.sh --zookeeper Snode1:2181 --topic message --delete


```

## Flink 安装与配置

+ 安装解压``Flink`
+ 修改环境变量

### 修改flink配置文件

+ 配置`flink/conf/flink-conf.yaml`

```
jobmanager.rpc.address: Snode1
taskmanager.heap.size: 6144
taskmanager.numberOfTaskSlots: 4
parallelism.default:8
```

+ 配置`slaves`

```
Snode2
Snode3
```

+ 配置`master`

```
Snnode1:8081
```

+ yarn -site.xml

  ```
  <configuration>
      <!-- 指定yarn的shuffle技术-->
      <property>
          <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
      </property>
      <!-- 指定resourcemanager的主机名-->
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>Snode1</value>
      </property>
  
  <!-- Site specific YARN configuration properties -->
  
  <property>
      <description>Whether to enable log aggregation</description>
      <name>yarn.log-aggregation-enable</name>
      <value>true</value>
  </property>
  <property>
      <name>yarn.log.server.url</name>
      <value>http://Snode1:19888/jobhistory/logs</value>
  </property>
  <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认
  是 true -->
  <property>
       <name>yarn.nodemanager.pmem-check-enabled</name>
       <value>false</value>
  </property>
  
  <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
  是 true -->
  <property>
       <name>yarn.nodemanager.vmem-check-enabled</name>
       <value>false</value>
  </property>
  <property>
  
  <name>yarn.nodemanager.pmem-check-enabled</name>
  
      <value>false</value>
  
  </property>
  
  <property>
  
       <name>yarn.nodemanager.vmem-check-enabled</name>
  
       <value>false</value>
  
  </property>
  </configuration>
  
  ```

  
