---
title: RocketMQ 集群 理论  搭建
#文章创建日期
date:  2019-01-15 20:00:00
#文章分类
categories: RocketMQ  MQ
#文章关键字
keywords: RocketMQ MQ
#文章描述
description: RocketMQ 笔记
#文章标籤 
tags: 
- RocketMQ
- MQ 
- 学习
#文章顶部图片
top_img: 
#文章缩略图
cover: /img/background/RocketMQ.jpg
---

## 五、集群搭建理论

**针对broker集群**

![分布式消息队列RocketMQ17.png](/img/RocketMQ/分布式消息队列RocketMQ17.png)

---


### 1 数据复制与刷盘策略

**刷盘：内存到磁盘**

![分布式消息队列RocketMQ18.png](/img/RocketMQ/分布式消息队列RocketMQ18.png)

- 复制策略
    复制策略是Broker的Master与Slave间的数据同步方式。分为同步复制与异步复制：
     **同步复制** ：消息写入master后，master会等待slave同步数据成功后才向producer返回成功ACK
     **异步复制** ：消息写入master后，master立即向producer返回成功ACK，无需等待slave同步数据成功

> 异步复制策略会降低系统的写入延迟，RT变小，提高了系统的吞吐量

---


- 刷盘策略
   刷盘策略指的是broker中消息的**落盘方式**，即**消息发送到broker内存后消息持久化到磁盘的方式**。分为 **同步刷盘与异步刷盘** ：

   - **同步刷盘** ：当消息持久化到broker的磁盘后才算是消息写入成功。
   - **异步刷盘** ：当消息写入到broker的内存后即表示消息写入成功，无需等待消息持久化到磁盘。

 > 1 ）异步刷盘策略会降低系统的写入延迟，RT变小，提高了系统的吞吐量
 > 2 ）消息写入到Broker的内存，一般是写入到了PageCache
 > 3 ）对于异步 刷盘策略，消息会写入到PageCache后立即返回成功ACK。但并不会立即做落盘操作，而是当PageCache到达一定量时会自动进行落盘。

---

### 2 Broker集群模式

根据Broker集群中各个节点间关系的不同，Broker集群可以分为以下几类：

- 单Master
   只有一个broker（其本质上就不能称为集群）。这种方式也只能是在测试时使用，生产环境下不能使用，因为存在单点问题。

- 多Master
   broker集群仅由多个master构成，不存在Slave。同一Topic的各个Queue会平均分布在各个master节点上。

    - **优点** ：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为**RAID10**时，即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢），性能最高；
    - **缺点** ：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅（不可消费），消息实时性会受到影响。

> 以上优点的前提是，这些Master都配置了RAID磁盘阵列。如果没有配置，一旦出现某Master宕机，则会发生大量消息丢失的情况。

---

- 多Master多Slave模式-异步复制

    - broker集群由多个master构成，每个master又配置了多个slave（在配置了RAID磁盘阵列的情况下，一个master一般配置一个slave即可）。master与slave的关系是主备关系，即master负责处理消息的读写请求，而slave仅负责消息的备份与master宕机后的角色切换。

    - 异步复制即前面所讲的复制策略中的异步复制策略，即消息写入master成功后，master立即向producer返回成功ACK，无需等待slave同步数据成功。

    - 该模式的最大特点之一是，当master宕机后slave能够**自动切换**为master。不过由于slave从master的同步具有短暂的延迟（毫秒级），所以当master宕机后，这种异步复制方式可能会存在少量消息的丢失问题。

> Slave从Master同步的延迟越短，其可能丢失的消息就越少，对于Master的RAID磁盘阵列；
> 若使用的也是异步复制策略，同样也存在延迟问题，同样也可能会丢失消息。
> 但RAID阵列的秘诀是微秒级的（因为是由硬盘支持的），所以其丢失的数据量会更少。

---

- 多Master多Slave模式-同步双写

  - 该模式是**多Master多Slave模式**的**同步复制**实现。所谓**同步双写**，指的是消息写入master成功后，master会等待slave同步数据成功后才向producer返回成功ACK，即master与slave都要写入成功后才会返回成功ACK，也即**双写**。
  该模式与异步复制模式相比，优点是消息的**安全性更高**，不存在消息丢失的情况。但单个消息的RT(响应时间)略 高，从而导致性能要略低（大约低10%）。
  该模式存在一个大的问题：**对于目前的版本，Master宕机后，Slave不会自动切换到Master（致命问题）**

---

- 最佳实践
  - 一般会为Master配置RAID10磁盘阵列，然后再为其配置一个Slave。即利用了RAID10磁盘阵列的高效、安全性，又解决了可能会影响订阅的问题。

> 1）RAID磁盘阵列的效率要高于Master-Slave集群。因为RAID是硬件支持的。也正因为如此，所以RAID阵列的搭建成本较高。
> 2）多Master+RAID阵列，与多Master多Slave集群的区别是什么？
>  - 多Master+RAID阵列，其仅仅可以保证数据不丢失，即不影响消息写入，但其可能会影响到消息的订阅。但其执行效率要远高于**多Master多Slave集群**
>  - 多Master多Slave集群，其不仅可以保证数据不丢失，也不会影响消息写入。其运行效率要低于多Master+RAID阵列

---

## 六、磁盘阵列RAID（补充）

### 1 RAID历史

1988 年美国加州大学伯克利分校的 D. A. Patterson 教授等首次在论文 “A Case of Redundant Array ofInexpensive Disks” 中提出了 RAID 概念 ，即**廉价冗余磁盘阵列（ Redundant Array of InexpensiveDisks ）**。

由于当时大容量磁盘比较昂贵， RAID 的基本思想是将多个容量较小、相对廉价的磁盘进行有机组合，从而以较低的成本获得与昂贵大容量磁盘相当的容量、性能、可靠性。随着磁盘成本和价格的不断降低， “廉价” 已经毫无意义。因此， RAID 咨询委员会（ RAID Advisory Board, RAB ）决定用“ 独立 ” 替代 “ 廉价 ” ，于时 RAID 变成了**独立磁盘冗余阵列（ Redundant Array of IndependentDisks ）**。但这仅仅是名称的变化，实质内容没有改变。

---

### 2 RAID等级

RAID 这种设计思想很快被业界接纳， RAID 技术作为**高性能、高可靠的存储技术**，得到了非常广泛的应用。 RAID 主要利用**镜像、数据条带和数据校验**三种技术来获取高性能、可靠性、容错能力和扩展性，根据对这三种技术的使用策略和组合架构，可以把 RAID 分为不同的等级，以满足不同数据应用的需求。

D. A. Patterson 等的论文中定义了 RAID0 ~ RAID6 原始 RAID 等级。随后存储厂商又不断推出 RAID7、 RAID10、RAID01 、 RAID50 、 RAID53 、 RAID100 等 RAID 等级，但这些并无统一的标准。目前业界与学术界公认的标准是 RAID0 ~ RAID6 ，而在实际应用领域中使用最多的 RAID 等级是 RAID0 、RAID1 、 RAID3 、 RAID5 、 RAID6 和 RAID10。

RAID每一个等级代表一种实现方法和技术，等级之间并无高低之分。在实际应用中，应当根据用户的数据应用特点，综合考虑可用性、性能和成本来选择合适的RAID 等级，以及具体的实现方式。

---


### 3 关键技术

- 镜像技术
   镜像技术是一种冗余技术，为磁盘提供数据备份功能，防止磁盘发生故障而造成数据丢失。对于 RAID而言，采用镜像技术最典型地的用法就是，同时在磁盘阵列中产生两个完全相同的数据副本，并且分布在两个不同的磁盘上。镜像提供了完全的数据冗余能力，当一个数据副本失效不可用时，外部系统仍可正常访问另一副本，不会对应用系统运行和性能产生影响。而且，镜像不需要额外的计算和校验，故障修复非常快，直接复制即可。镜像技术可以从多个副本进行并发读取数据，提供更高的读 I/O 性能，但不能并行写数据，写多个副本通常会导致一定的 I/O 性能下降。

镜像技术提供了非常高的数据安全性，其代价也是非常昂贵的，需要至少双倍的存储空间。高成本限制了镜像的广泛应用，主要应用于至关重要的数据保护，这种场合下的数据丢失可能会造成非常巨大的损失。

- 数据条带技术
   将一个数据拆开，并行写给多个磁盘中；数据条带化技术是一种自动将 I/O操作负载均衡到多个物理磁盘上的技术。更具体地说就是，将一块连续的数据分成很多小部分并把它们分别存储到不同磁盘上。这就能使多个进程可以并发访问数据的多个不同部分，从而获得最大程度上的 I/O并行能力，极大地提升 性能。

- 数据校验技术
   数据校验技术是指， RAID 要在写入数据的同时进行校验计算，并将得到的校验数据存储在 RAID 成员磁盘中。校验数据可以集中保存在某个磁盘或分散存储在多个不同磁盘中。当其中一部分数据出错时，就可以对剩余数据和校验数据进行反校验计算重建丢失的数据。

数据校验技术相对于镜像技术的优势在于节省大量开销，但由于每次数据读写都要进行大量的校验运算，对计算机的运算速度要求很高，且必须使用硬件 RAID 控制器。在数据重建恢复方面，检验技术比镜像技术复杂得多且慢得多。

---


### 4 RAID分类

从实现角度看， RAID 主要分为软 RAID、硬 RAID 以及混合 RAID 三种。

- **软 RAID**
   所有功能均有操作系统和 CPU 来完成，没有独立的 RAID 控制处理芯片和 I/O 处理芯片，效率自然最低。软件完成

- **硬 RAID**
   配备了专门的 RAID 控制处理芯片和 I/O 处理芯片以及阵列缓冲，不占用 CPU 资源。效率很高，但成本也很高。硬件完成

- **混合 RAID**
   具备 RAID 控制处理芯片，但没有专门的I/O 处理芯片，需要 CPU 和驱动程序来完成。性能和成本在软RAID 和硬 RAID 之间。软硬完成

---


### 5 常见RAID等级详解

- JBOD

![分布式消息队列RocketMQ19.png](/img/RocketMQ/分布式消息队列RocketMQ19.png)

**JBOD** ，Just a Bunch of Disks，磁盘簇。表示一个没有控制软件提供协调控制的磁盘集合，这是 RAID区别与 JBOD 的主要因素。 JBOD 将**多个物理磁盘串联起来**，提供一个巨大的**逻辑磁盘**。

JBOD 的数据存放机制是由第一块磁盘开始按顺序往后存储，当前磁盘存储空间用完后，再依次往后面的磁盘存储数据。 JBOD 存储性能**完全等同于单块磁盘**，而且也不提供数据安全保护。

> 多块磁盘组成的逻辑磁盘；
> 其只是简单提供一种扩展存储空间的机制，JBOD可用存储容量等于所有成员磁盘的存储空间之和

JBOD 常指磁盘柜，而不论其是否提供 RAID 功能。不过，JBOD并非官方术语，官方称为Spanning。

---

- RAID0

![分布式消息队列RocketMQ20.png](/img/RocketMQ/分布式消息队列RocketMQ20.png)

RAID0 是 一种简单的、无数据校验的**数据条带化技术**。实际上不是一种真正的 RAID ，因为它并不提供任何形式的冗余策略 。 RAID0 将所在磁盘条带化后组成大容量的存储空间，将数据分散存储在所有磁盘中，以独立访问方式实现多块磁盘的**并读访问**。

理论上讲，一个由 n 块磁盘组成的 RAID0 ，它的**读写性能是单个磁盘性能的 n 倍**, 但由于总线带宽等多种因素的限制， 实际的 性能提升低于理论值。由于可以并发执行 I/O 操作，总线带宽得到充分利用。再加上不需要进行数据校验，**RAID0 的性能在所有 RAID 等级中是最高的**。

RAID0 具有低成本、高读写性能、 100% 的高存储空间利用率等优点，但是它不提供数据冗余保护，一旦数据损坏，将无法恢复。

**应用场景**：对数据的顺序读写要求不高，对数据的安全性和可靠性要求不高，但对系统性能要求很高的场景。

> RAID0与JBOD相同点：
> 1）存储容量：都是成员磁盘容量总和
> 2）磁盘利用率，都是100%，即都没有做任何的数据冗余备份
> RAID0与JBOD不同点：
> JBOD：数据是顺序存放的，一个磁盘存满后才会开始存放到下一个磁盘
> RAID：各个磁盘中的数据写入是并行的，是通过数据条带技术写入的。其读写性能是JBOD的n倍

---


- RAID1

![分布式消息队列RocketMQ21.png](/img/RocketMQ/分布式消息队列RocketMQ21.png)

RAID1 就是一种**镜像技术**，它将数据完全一致地分别写到工作磁盘和镜像磁盘，它的磁盘空间利用率为 50% 。 RAID1 在数据写入时，响应时间会有所影响，但是读数据的时候没有影响。RAID1提供了最佳的数据保护，一旦工作磁盘发生故障，系统将自动切换到镜像磁盘，不会影响使用。

RAID1是为了增强数据安全性使两块磁盘数据呈现完全镜像，从而达到安全性好、技术简单、管理方便。 RAID1 拥有完全容错的能力，但实现成本高。

**应用场景**：对顺序读写性能要求较高，或对数据安全性要求较高的场景。

---

- RAID10

![分布式消息队列RocketMQ22.png](/img/RocketMQ/分布式消息队列RocketMQ22.png)

RAID10是一个RAID1与RAID0的组合体，所以**它继承了RAID0的快速和RAID1的安全**。

简单来说就是，**先做条带，再做镜像**。发即将进来的数据先分散到不同的磁盘，再将磁盘中的数据做镜像。

---


- RAID01

![分布式消息队列RocketMQ23.png](/img/RocketMQ/分布式消息队列RocketMQ23.png)


RAID01是一个RAID0与RAID1的组合体，所以**它继承了RAID0的快速和RAID1的安全**。

简单来说就是，**先做镜像,再做条带**。即将进来的数据先做镜像，再将镜像数据写入到与之前数据不同的磁盘，即再做条带。

> RAID10要比RAID01的容错率再高，所以生产环境下一般是不使用RAID01的。

---

## 七、集群搭建实践

### 1 集群架构

这里要搭建一个双主双从异步复制的Broker集群。为了方便，这里使用了两台主机来完成集群的搭建。
这两台主机的功能与broker角色分配如下表。

| 序号|	主机名/IP	|         IP	    |            功 能	       |        BROKER角色        |
|:----:|:----:        |     :----:      |       :----:              |        :----:           |
|  1  |	 s4	        |   192.168.109.104	|    NameServer + Broker   |	    Master1 + Slave2  |
|  2  |	 s5	        |   192.168.109.105	|    NameServer + Broker   |	    Master2 + Slave1  | 

---


### 2 克隆生成rocketmqOS1

克隆rocketmqOS主机，并修改配置。指定主机名为rocketmqOS1。

```rocketmq
IP
/etc/sysconfig/network-scripts/ifcfg-ens33
```

设置主机名：
```rocketmq
hostnamectl set-hostname 你要设置的主机名
```
设置映射：

```rocketmq
vim /etc/hosts
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 s4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

reboot #重启linux服务器
```
---

### 3 修改rocketmqOS1配置文件

- 配置文件位置

要修改的配置文件在rocketMQ解压目录的conf/2m-2s-async目录中
![a911f2d7203744a7ac6f5a03b72b2831.png](/img/RocketMQ/a911f2d7203744a7ac6f5a03b72b2831.png)

![分布式消息队列RocketMQ24.png](/img/RocketMQ/分布式消息队列RocketMQ24.png)

- 修改broker-a.properties----------Master机
  - 将该配置文件内容修改为如下：
```rocketmq
# 指定整个broker集群的名称，或者说是RocketMQ集群的名称
brokerClusterName=DefaultCluster
# 指定master-slave集群的名称。一个RocketMQ集群可以包含多个master-slave集群
brokerName=broker-a
# master的brokerId为0
brokerId=0
# 指定删除消息存储过期文件的时间为凌晨4点
deleteWhen=04
# 指定未发生更新的消息存储文件的保留时长为48小时，48小时后过期，将会被删除
fileReservedTime=48
# 指定当前broker为异步复制master
brokerRole=ASYNC_MASTER
# 指定刷盘策略为异步刷盘
flushDiskType=ASYNC_FLUSH
# 指定Name Server的地址
namesrvAddr=192.168.109.104:9876;192.168.109.105:9876
```

- 修改broker-b-s.properties-------Salve机
  - 将该配置文件内容修改为如下
```rocketmq
brokerClusterName=DefaultCluster
# 指定这是另外一个master-slave集群
brokerName=broker-b
# slave的brokerId为非0
brokerId=1
deleteWhen=04
fileReservedTime=48
# 指定当前broker为slave
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=192.168.109.104:9876;192.168.109.105:9876
# 指定Broker对外提供服务的端口，即Broker与producer与consumer通信的端口。默认10911。由于当前主机同时充当着master1与slave2，而前面的master1使用的是默认端口。这里需要将这两个端口加以区分，以区分出master1与slave2
listenPort=11911
# 指定消息存储相关的路径。默认路径为~/store目录。由于当前主机同时充当着master1与slave2，master1使用的是默认路径，这里就需要再指定一个不同路径
storePathRootDir=~/store-s
storePathCommitLog=~/store-s/commitlog
storePathConsumeQueue=~/store-s/consumequeue
storePathIndex=~/store-s/index
storeCheckpoint=~/store-s/checkpoint
abortFile=~/store-s/abort

```

- 其他配置
  - 除了以上配置外，这些配置文件中还可以设置其它属性。
 ```rocketmq
 #指定整个broker集群的名称，或者说是RocketMQ集群的名称
brokerClusterName=rocket-MS
#指定master-slave集群的名称。一个RocketMQ集群可以包含多个master-slave集群
brokerName=broker-a
#0 表示 Master，>0 表示 Slave
brokerId=0
#多个nameServer地址，分号分割
namesrvAddr=nameserver1:9876;nameserver2:9876
#默认为新建Topic所创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议生产环境中关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议生产环境中关闭
autoCreateSubscriptionGroup=true
#Broker对外提供服务的端口，即Broker与producer与consumer通信的端口
listenPort=10911
#HA高可用监听端口，即Master与Slave间通信的端口，默认值为listenPort+1
haListenPort=10912
#指定删除消息存储过期文件的时间为凌晨4点
deleteWhen=04
#指定未发生更新的消息存储文件的保留时长为48小时，48小时后过期，将会被删除
fileReservedTime=48
#指定commitLog目录中每个文件的大小，默认1G
mapedFileSizeCommitLog=1073741824
#指定ConsumeQueue的每个Topic的每个Queue文件中可以存放的消息数量，默认30w条
mapedFileSizeConsumeQueue=300000
#在清除过期文件时，如果该文件被其他线程所占用（引用数大于0，比如读取消息），此时会阻止此次删除任务，同时在第一次试图删除该文件时记录当前时间戳。该属性则表示从第一次拒绝删除后开始计时，该文件最多可以保留的时长。在此时间内若引用数仍不为0，则删除仍会被拒绝。不过时间到后，文件将被强制删除
destroyMapedFileIntervalForcibly=120000
#指定commitlog、consumequeue所在磁盘分区的最大使用率，超过该值，则需立即清除过期文
件
diskMaxUsedSpaceRatio=88
#指定store目录的路径，默认在当前用户主目录中
storePathRootDir=/usr/local/rocketmq-all-4.5.0/store
#commitLog目录路径
storePathCommitLog=/usr/local/rocketmq-all-4.5.0/store/commitlog
#consumeueue目录路径
storePathConsumeQueue=/usr/local/rocketmq-all-4.5.0/store/consumequeue
#index目录路径
storePathIndex=/usr/local/rocketmq-all-4.5.0/store/index
#checkpoint文件路径
storeCheckpoint=/usr/local/rocketmq-all-4.5.0/store/checkpoint
#abort文件路径
abortFile=/usr/local/rocketmq-all-4.5.0/store/abort
#指定消息的最大大小
maxMessageSize=65536
#Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写Master
# - SLAVE
brokerRole=SYNC_MASTER
#刷盘策略
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
#发消息线程池数量
sendMessageThreadPoolNums=128
#拉消息线程池数量
pullMessageThreadPoolNums=128
#强制指定本机IP，需要根据每台机器进行修改。官方介绍可为空，系统默认自动识别，但多网卡时IP地址可能读取错误
brokerIP1=192.168.3.105
 ```

 ---

### 4 克隆生成rocketmqOS2

克隆rocketmqOS1主机，并修改配置。指定主机名为rocketmqOS2。

### 5 修改rocketmqOS2配置文件

对于rocketmqOS2主机，同样需要修改rocketMQ解压目录的conf目录的子目录**2m-2s-async**中的两个配置文件。

- 修改broker-b.properties-----Master机
  - 将该配置文件内容修改为如下：
```rocketmq
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
namesrvAddr=192.168.109.104:9876;192.168.109.105:9876
```

- 修改broker-a-s.properties---------Salve机
  - 将该配置文件内容修改为如下：
```rocketmq
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=192.168.109.104:9876;192.168.109.105:9876
listenPort=11911
storePathRootDir=~/store-s
storePathCommitLog=~/store-s/commitlog
storePathConsumeQueue=~/store-s/consumequeue
storePathIndex=~/store-s/index
storeCheckpoint=~/store-s/checkpoint
abortFile=~/store-s/abort
```

---

### 6 启动服务器

- 启动NameServer集群
  - 分别启动rocketmqOS1与rocketmqOS2两个主机中的NameServer。启动命令完全相同。
```rocketmq
nohup sh bin/mqnamesrv & tail -f ~/logs/rocketmqlogs/namesrv.log
```
![54f9beeb340d4067ac7aedbefe7a7136.png](/img/RocketMQ/54f9beeb340d4067ac7aedbefe7a7136.png)

- 启动两个Master
  - 分别启动rocketmqOS1与rocketmqOS2两个主机中的broker master。**注意，它们指定所要加载的配置文件是不同的**。
![分布式消息队列RocketMQ23.png](/img/RocketMQ/分布式消息队列RocketMQ23.png)
```rocketmq
nohup sh bin/mqbroker -c conf/2m-2s-async/broker-a.properties &
tail -f ~/logs/rocketmqlogs/broker.log

nohup sh bin/mqbroker -c conf/2m-2s-async/broker-b.properties &
tail -f ~/logs/rocketmqlogs/broker.log
```

- 启动两个Slave
  - 分别启动s4与s5两个主机中的broker slave。**注意，它们指定所要加载的配置文件是不同的**。
  ```rocketmq
  nohup sh bin/mqbroker -c conf/2m-2s-async/broker-b-s.properties &
tail -f ~/logs/rocketmqlogs/broker.log

nohup sh bin/mqbroker -c conf/2m-2s-async/broker-a-s.properties &
tail -f ~/logs/rocketmqlogs/broker.log
```
- 两台机器s4/s5，两个小集群，s4中broker-a|broker-b-s，s5中broker-b|broker-a-s组成一个rocket集群


![a2702f6d230343288cdb687c44155f18.png](/img/RocketMQ/a2702f6d230343288cdb687c44155f18.png)

---

## 八、mqadmin命令

在mq解压目录的bin目录下有一个mqadmin命令，该命令是一个运维指令，用于对mq的主题，集群，broker 等信息进行管理。

### 1 修改bin/tools.sh

在运行mqadmin命令之前，先要修改mq解压目录下bin/tools.sh配置的JDK的ext目录位置。本机的ext目录在/usr/local/java/jdk1.8.0_161/jre/lib/ext。

使用vim命令打开tools.sh文件，并在JAVA_OPT配置的-Djava.ext.dirs这一行的后面添加ext的路径。

![970f7318a92740ff9b241803b9bad13d.png](/img/RocketMQ/970f7318a92740ff9b241803b9bad13d.png)

![分布式消息队列RocketMQ25.png](/img/RocketMQ/分布式消息队列RocketMQ25.png)

```rocketmq
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn256m -
XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m"
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib:${JAVA_HOME}/jre/lib/ext:${JAVA_HOME}/lib/ext:/usr/local/java/jdk1.8.0_161/jre/lib/ext"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"
```
---

### 2 运行mqadmin

直接运行该命令，可以看到其可以添加的commands。通过这些commands可以完成很多的功能。
```rocketmq
[root@s4 bin]# ./mqadmin
The most commonly used mqadmin commands are:
   updateTopic          Update or create topic
   deleteTopic          Delete topic from broker and NameServer.
   updateSubGroup       Update or create subscription group
   deleteSubGroup       Delete subscription group from broker.
   updateBrokerConfig   Update broker's config
   updateTopicPerm      Update topic perm
   topicRoute           Examine topic route info
   topicStatus          Examine topic Status info
   topicClusterList     get cluster info for topic
   brokerStatus         Fetch broker runtime status data
   queryMsgById         Query Message by Id
   queryMsgByKey        Query Message by Key
   queryMsgByUniqueKey  Query Message by Unique key
   queryMsgByOffset     Query Message by offset
   QueryMsgTraceById    query a message trace
   printMsg             Print Message Detail
   printMsgByQueue      Print Message Detail
   sendMsgStatus        send msg to broker.
   brokerConsumeStats   Fetch broker consume stats data
   producerConnection   Query producer's socket connection and client version
   consumerConnection   Query consumer's socket connection, client version and subscription
   consumerProgress     Query consumers's progress, speed
   consumerStatus       Query consumer's internal data structure
   cloneGroupOffset     clone offset from other group.
   clusterList          List all of clusters
   topicList            Fetch all topic list from name server
   updateKvConfig       Create or update KV config.
   deleteKvConfig       Delete KV config.
   wipeWritePerm        Wipe write perm of broker in all name server
   resetOffsetByTime    Reset consumer offset by timestamp(without client restart).
   updateOrderConf      Create or update or delete order conf
   cleanExpiredCQ       Clean expired ConsumeQueue on broker.
   cleanUnusedTopic     Clean unused topic on broker.
   startMonitoring      Start Monitoring
   statsAll             Topic and Consumer tps stats
   allocateMQ           Allocate MQ
   checkMsgSendRT       check message send response time
   clusterRT            List All clusters Message Send RT
   getNamesrvConfig     Get configs of name server.
   updateNamesrvConfig  Update configs of name server.
   getBrokerConfig      Get broker config by cluster or special broker!
   queryCq              Query cq command.
   sendMessage          Send a message
   consumeMessage       Consume message
   updateAclConfig      Update acl config yaml file in broker
   deleteAccessConfig   Delete Acl Config Account in broker
   clusterAclConfigVersion List all of acl config version information in cluster
   updateGlobalWhiteAddr Update global white address for acl Config File in broker
   getAccessConfigSubCommand List all of acl config information in cluster

See 'mqadmin help <command>' for more information on a specific command.
```
---

### 3 该命令的官网详解

- 该命令在官网中有详细的用法解释。

https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md

![分布式消息队列RocketMQ26.png](/img/RocketMQ/分布式消息队列RocketMQ26.png)

![分布式消息队列RocketMQ27.png](/img/RocketMQ/分布式消息队列RocketMQ27.png)