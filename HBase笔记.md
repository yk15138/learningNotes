# HBase笔记

## 1、详述hbase的架构

![](https://pic002.cnblogs.com/images/2012/79891/2012060400561367.jpg)

## 2、详述hbase的组件及其作用

- **Client**

  > 使用HBase RPC机制与HMaster和HRegionServer进行通信 
  > Client与HMaster进行通信进行管理类操作 
  > Client与HRegionServer进行数据读写类操作

- **Zookeeper**

  > Zookeeper Quorum存储-ROOT-表地址、HMaster地址 
  > HRegionServer把自己以Ephedral方式注册到Zookeeper中，HMaster随时感知各个HRegionServer的健康状况 
  > Zookeeper避免HMaster单点问题 

- **HMaster**

  > HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的Master Election机制保证总有一个Master在运行 
  > 主要负责Table和Region的管理工作： 
  > 　　1 管理用户对表的增删改查操作 
  > 　　2 管理HRegionServer的负载均衡，调整Region分布 
  > 　　3 Region Split后，负责新Region的分布 
  > 　　4 在HRegionServer停机后，负责失效HRegionServer上Region迁移 

- **HRegionServer**

  > HBase中最核心的模块，主要负责响应用户I/O请求，向HDFS文件系统中读写数据
  >
  > HRegionServer管理一些列HRegion对象； 
  > 每个HRegion对应Table中一个Region，HRegion由多个HStore组成； 
  > 每个HStore对应Table中一个Column Family的存储； 
  > Column Family就是一个集中的存储单元，故将具有相同IO特性的Column放在一个Column Family会更高效

- **HStore**

  > HBase存储的核心。由MemStore和StoreFile组成。
  >
  > Client写入 -> 存入MemStore，一直到MemStore满 -> Flush成一个StoreFile，直至增长到一定阈值 -> 触发Compact合并操作 -> 多个StoreFile合并成一个StoreFile，同时进行版本合并和数据删除 -> 当StoreFiles Compact后，逐步形成越来越大的StoreFile -> 单个StoreFile大小超过一定阈值后，触发Split操作，把当前Region Split成2个Region，Region会下线，新Split出的2个孩子Region会被HMaster分配到相应的HRegionServer上，使得原先1个Region的压力得以分流到2个Region上。
  > 由此过程可知，HBase只是增加数据，有所得更新和删除操作，都是在Compact阶段做的，所以，用户写操作只需要进入到内存即可立即返回，从而保证I/O高性能。

- **HLog**

  > 引入HLog原因： 
  > 　　在分布式系统环境中，无法避免系统出错或者宕机，一旦HRegionServer意外退出，MemStore中的内存数据就会丢失，引入HLog就是防止这种情况 
  >
  > 工作机制： 
  > 　　每个HRegionServer中都会有一个HLog对象，HLog是一个实现Write Ahead Log的类，每次用户操作写入Memstore的同时，也会写一份数据到HLog文件，HLog文件定期会滚动出新，并删除旧的文件(已持久化到StoreFile中的数据)。当HRegionServer意外终止后，HMaster会通过Zookeeper感知，HMaster首先处理遗留的HLog文件，将不同region的log数据拆分，分别放到相应region目录下，然后再将失效的region重新分配，领取到这些region的HRegionServer在Load Region的过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后flush到StoreFiles，完成数据恢复。

## 3、详述hbase的读写流程

### 写流程

- 1、Client访问ZK，根据ROOT表获取meta表所在Region的位置信息，并将该位置信息写入Client Cache。 
  （注：为了加快数据访问速度，我们将元数据、Region位置等信息缓存在Client Cache中。）

- 2、Client读取meta表，再根据meta表中查询得到的Namespace、表名和RowKey等相关信息，获取将要写入Region的位置信息（此过程即Region三层定位，如下图），最后client端会将meta表写入Client Cache。 
- 3、Client向上一步HRegionServer发出写请求，HRegionServer先将操作和数据写入HLog（预写日志，Write Ahead Log，WAL），再将数据写入MemStore，并保持有序。 （联想：HDFS中也是如此，EditLog写入时机也是在真实读写之前发生
- 4、当MemStore的数据量超过阈值时，将数据溢写磁盘，生成一个StoreFile文件。 
  当Store中**StoreFile的数量**超过阈值时，将若干小StoreFile合并（Compact）为一个大StoreFile。 当Region中**最大Store的大小**超过阈值时，Region分裂（Split），等分成两个子Region。

### 读流程

- 1、获取将要读取Region的位置信息
- 2、Client向HRegionServer发出读请求
- 3、HRegionServer先从MemStore读取数据，如未找到，再从StoreFile中读取

## 4、Hbase集群搭建步骤

- 时间同步

```shell
nptdate npt1.aliyun.com
```

- 免秘钥

```
ssh-copy-id -r /root/.ssh/id_rsa.pub <ip>
```

- 把要把hadoop的hdfs-site.xml和core-site.xml 拷贝到hbase/conf下(注意：Hadoop路径可能不一样)

- 配置hbase环境变量

- 修改vi hbase-env.sh（JAVA_HOME路径以及true改为false）

```shell
#配置JAVA_HOME
export JAVA_HOME=/usr/java/jdkJAVA_HOME
#hbase不自己管理zk
export HBASE_MANAGES_ZK=false 
```

- 修改hbase-site.xml

```xml
<!-- master是你的主节点名，也可以写成ip -->
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://master:90000/hbase</value>
</property>
<!-- 指定hbase是分布式的 -->
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<!-- 指定zk的地址，多个用“,”分割 -->
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>master:2181,slave01:2181,slave02:2181</value>
    <!-- master，slave,slave是你的节点名，也可以写成ip,可以不写端口号 -->
</property>
```

- 修改regionservers设置regionservers的地址

```
node11
node12
node13
```

- 修改backup-masters设置hmaster的地址

```
node11
```

- 启动

```shell
start-hbase.sh
```



## 5、列出hbase的基本操作

- 看文件

```shell
# 本地
hbase hfile -p -f /user/home
# hdfs
hbase hfile
```



- 查看当前用户

```shell
whoami
```
-	查看有哪些表

```shell
list
```

- 创建表

```shell

```










## 6、列出hbase的java api基本操作



# 问题：

- hbase的增删改流程（从定位到hlog到memostore到flush到storefile，从内存到外存怎么存的什么，怎么标记过期。）

>在HBase 0.96之前，保留的默认版本数为3，但是在0.96中，更新版本已更改为1。

> ​	put,delete 在memostore中所有的操作都是累加，无论多少个副本都在内存。flush的时候会合并

> 一个delete会吧所有的'table','cf:name'删掉



- hbase的缓存
- hfile的存储结构
- hfile的压缩
- region和列族的分裂，合并
- flush的触发条件
