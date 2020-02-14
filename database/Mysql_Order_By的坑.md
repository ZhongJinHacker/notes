```mysql
CREATE TABLE `tt_transfer_container_pick_config` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `yard_code` varchar(11) NOT NULL DEFAULT '' COMMENT '场地代码',
	...
  `version` bigint(20) DEFAULT NULL COMMENT '版本号',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_yard_code` (`yard_code`) USING BTREE COMMENT '场院代码索引',
	...
  KEY `idx_create_time` (`create_time`) USING BTREE COMMENT '创建时间索引',
  KEY `idx_update_time` (`update_time`) USING BTREE COMMENT '更新时间索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='容器分拣配置表';
```

查询时出问题的就是这张表

出错的查询语句

```mysql
SELECT * 
FROM tt_transfer_container_pick_config 
WHERE 
    yard_code = '735VA' 
    AND 
    ...
    AND
    is_delete = 0 
    AND 
    invalid_time > NOW() 
    ORDER BY  update_time DESC
    LIMIT 1, 10;
```



==order by update_time DESC==咋一看没有问题，update_time 是每次都是由mysql自动更新的，感觉会一直递增



### 第一个问题：

#### 实际上如果在同一个事务里，批量插入多条数据，这里的update_time会==是一样的==，因为是事务结束时一齐从内存中写进数据库里的（用Excel导入且在同一个事务中测试得出的结论）

#### 



###由第一个问题引发的第二个问题（当然如果通过脚本直接改update_time为一样的，也会有这个问题）：

#### 就是==order by update_time DESC==时，mysql会用他自己的随机算法从中抽取你需要的量的数据给你，比如上述需要9条数据，匹配的数据里有100条，就会随机抽取9条数据给你，而由于mysql自己有缓存提速机制，会导致如果第一次拿到的数据不对，后面再用相同的sql查数据都是之前的错误数据，而且每次分页查（查其他分页）都是随机的，会有的页出现其他页存在重复数据，有的数据永远读不出来的现象



### 更正方法：在order by 中 加入主键 id 排序，这样就能保证所有的数据都可以在分页中查出来，并且各个分页的数据有序且不重复

```mysql
SELECT * 
FROM tt_transfer_container_pick_config 
WHERE 
    yard_code = '735VA' 
    AND 
    ...
    AND
    is_delete = 0 
    AND 
    invalid_time > NOW() 
    ORDER BY id ASC, update_time DESC
    LIMIT 1, 10;
```





