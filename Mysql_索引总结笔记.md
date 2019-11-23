# Mysql 索引总结

### 1. 聚簇索引

InnoDB 引擎使用的就是聚簇索引，就是主键的索引，是一种数据的存储方式。所有的数据都是存储在索引的叶子结点上（与MySAM 引擎不同，MySAM是传统方式），这样本质也是一种加速查找的方式，搜索索引就可以拿到想要的行所有的数据；不过对于不是顺序的插入（比如随机ID插入）不友好，会犹豫叶分裂、行移动重排的问题导致插入速度慢和数据碎片；全表扫描相对MySAM也会慢一些；具体《高性能Mysql》P163

### 2. 二级索引

非聚簇索引都是二级索引，即非主键索引就是二级索引；二级索引叶子节点保存的是本字段的值和主键id值，查询其他内容还需要拿着主键id再去查聚簇索引，就是查询了两次B+Tree，相对会慢一点

### 3. 覆盖索引

定义： 如果一个索引包含（或者说覆盖）所有需要查询的字段的值，我们称这种索引为“覆盖索引”。

覆盖索引可以极大提高性能，即只有一次B+Tree查询就搞定了

***在EXPLAIN时，如果在Extra 中开到“Using Index”，说明这条SQL利用到了覆盖索引***

比如：

```mysql
CREATE TABLE IF NOT EXISTS `user`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `name` VARCHAR(100) NOT NULL,
   `sex` VARCHAR(40) NOT NULL,
   `birthday` DATE,
   PRIMARY KEY ( `id` )
   KEY `idx_name` (`name`) USING BTREE,
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

```mysql
SELECT name FROM user
```

即可以直接使用到覆盖索引，因为name字段的索引树里本身就有name的值，就不需要去聚簇索引里再查了