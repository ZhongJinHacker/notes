```sql
CREATE TABLE `tt_transfer_assemble_diffuse_plan_info` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `yard_code` varchar(11) NOT NULL DEFAULT '' COMMENT '中转场代码',
  `situation` varchar(11) NOT NULL DEFAULT '' COMMENT '集散情况',
  `assemble_plan` varchar(20) DEFAULT NULL COMMENT '集货班次',
  `diffuse_plan` varchar(20) DEFAULT NULL COMMENT '散货班次',
  `start_time` varchar(11) NOT NULL DEFAULT '' COMMENT '开始时间',
  `end_time` varchar(11) NOT NULL DEFAULT '' COMMENT '结束时间',
  `diffuse_pick_partition` varchar(600) DEFAULT '' COMMENT '散货分拣片区',
  `operator` varchar(32) NOT NULL DEFAULT '' COMMENT '操作人',
  `create_by` varchar(10) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(10) DEFAULT NULL COMMENT '更新者',
  `delete_flg` tinyint(1) DEFAULT NULL COMMENT '是否删除{0:否,1:是}',
  `version` bigint(20) DEFAULT NULL COMMENT '版本号',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_unique` (`yard_code`,`situation`,`assemble_plan`,`diffuse_plan`) USING BTREE COMMENT '联合唯一索引',
  KEY `idx_create_time` (`create_time`) USING BTREE COMMENT '创建时间索引',
  KEY `idx_update_time` (`update_time`) USING BTREE COMMENT '更新时间索引'
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COMMENT='集散班次信息表';
```

唯一索引为联合唯一索引，如果这四个字段中有null 的话，是可以重复的，因为唯一索引允许null重复

比如对于这张表可以插入多条

```sql
537W J NULL NULL
537W J NULL NULL

537W J JIANG NULL
537W J JIANG NULL
```

