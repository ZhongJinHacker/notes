

#### Mysql 添加字段 例子

 ```mysql
ALTER TABLE tt_transfer_container_pick_config ADD COLUMN container_pick_station VARCHAR(11) DEFAULT NULL COMMENT '操作岗位(容器分拣)' AFTER station_no;
 ```



#### Mysql 修改字段 例子

```mysql
ALTER TABLE tt_transfer_container_pick_config MODIFY COLUMN station_no VARCHAR(50) COMMENT '操作岗位(装车)';

```

