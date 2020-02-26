#### kafka逻辑架构

![](/Users/Jiangzhongjin/Documents/CodeDir/githubDir/my/notes/kafka/pic/kafka_architecture.png)



producer（生产者）采用推数据方式将消息发送到Broker

每条消息都会顺序追加到分区中，顺序写入磁盘，保障吞吐率



#### 分区（Partition）

消息都是被发送到Topic，本质是一个目录，而Topic是由一些Partition Logs（分区日志）组成，组织结构如下：

![](/Users/Jiangzhongjin/Documents/CodeDir/githubDir/my/notes/kafka/pic/kafka_partition.png)

委婉待续