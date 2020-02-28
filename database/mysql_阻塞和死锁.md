### 什么是阻塞

#### 由于不同锁之间的兼容关系，造成一个事务需要等待另一个事务释放其所占用的资源的现象 称为 阻塞



### 如何发现阻塞

`mysql_8.0`

```mysql
SELECT waiting_pid as '被阻塞的线程',
       waiting_query as '被阻塞的SQL',
       blocking_pid as '阻塞线程',
       blocking_query as '阻塞SQL',
       waiting_age as '阻塞时间',
       sql_kill_blocking_query as '建议操作'
FROM
		sys.innodb_lock_waits
WHERE
		( UNIX_TIMESTAMP() - UNIX_TIMESTAMPP(wait_started) ) > 30;
```

这里的 30 是三十秒的意思



### 如何处理阻塞

1. 终止占用资源的事务（治标不治本）
2. 优化占用资源事务的SQL，使其尽快释放资源

------



### 什么是死锁

#### 并行执行的多个事务相互之间占有了对方所需要的资源



#### 比如:

##### 事务一

```mysql
BEGIN;
Update course set score = 9.7 where course_id = 35;
update user set score = score - 10 where user_id = 10;
```

##### 事务二

```mysql
BEGIN;
update user set score = score + 10 where user_id = 10;
Update course set score = 9.8 where course_id = 35;
```



==由于事务一和事务二 where条件一致，在一定的条件下会导致死锁，比如事务一在执行第一条sql时对该资源加排他锁，同时事务二对另一资源加上了排他锁，当两个事务在执行它们的第二条SQL时，会分别等待对方的排他锁释放，导致死锁发生==

#### 

#### 如何发现死锁

##### 将死锁记录到错误日志中

```mysql
set global innodb_print_all_deadlocks = on;
```



##### 死锁日志例子(整理后)：

```shell
TRANSACTION 1704, ACTIVE 119 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 2 row lock(s), undo log entries 1
MYSQL thread id 12, OS thread handle 14025222817xxxxx, query id 170 localhost root update
update imc_user set score = score + 10 where user_id = 10

```

==这里记录的可能不是导致产生死锁的那个SQL， 而是事务中发生死锁时正在执行的那个SQL，所以还是根据这个线索去程序中寻找问题的症结所在==



### 如何处理事务的死锁

##### 1. mysql 在发生死锁时会自行回滚占用资源少的事务（但这样就影响到了业务，治标不治本）

#### 2. 将发现的造成死锁的事务代码按相同的顺序去占用资源，资源可以将死锁降为阻塞



