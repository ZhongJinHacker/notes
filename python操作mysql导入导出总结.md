python 操作 mysql 导入导出



今天使用python 来导出数据库的一些表及数据

下面是执行正确的代码

```python
return '/usr/local/mysql/bin/mysqldump --column-statistics=0 --compress --set-gtid-purged=OFF  -h ' + SRC_DB_HOST\
           + ' -u' + SRC_DB_USERNAME + ' -p' + SRC_DB_PASSWORD\
           + ' --databases ' + SRC_DB_NAME + ' --tables ' + tables + ' > ' + SQL_FILE_PATH
```



由于我本地的mysqldump是8.0以上的，而目标mysql是5.6.17的，所以出了一些问题

最开始的执行命令是

```python
'/usr/local/mysql/bin/mysqldump  -h ' + SRC_DB_HOST\
           + ' -u' + SRC_DB_USERNAME + ' -p' + SRC_DB_PASSWORD\
           + ' --databases ' + SRC_DB_NAME + ' --tables ' + tables + ' > ' + SQL_FILE_PATH
```



没有

```python
--column-statistics=0 --compress --set-gtid-purged=OFF
```

导致导出总是报错



这里记录一些这几个参数的作用

[1] ==--column-statistics=0==

这是由于使用8.0以上的mysqldump导致的

mysqldump 8.0 以上默认打开了column-statistics 导致的

```json
错误提示：Couldn't execute SELECT COLUMN_NAME...

错误原因: 新版的mysqldump默认启用了一个新标志，通过 --column-statistics=0 来禁用他
```

[2] --set-gtid-purged=OFF

这个标志是用来标识（主从关系）在master上恢复数据，还是在slave上恢复数据



我们备份，就是可能需要拿来进行恢复，是在master上恢复，还是slave上恢复。

如果是在master上进行恢复，那么就需要生成对应的gtid,所以需要使用set-gtid-purged=off

如果是在slave上进行恢复，那么不需要生成对应的gtid，所以需要使用set-gtid-purged=on



GTID是5.6以后，加入了全局事务 ID (GTID) 来强化数据库的主备一致性，故障恢复，以及容错能力