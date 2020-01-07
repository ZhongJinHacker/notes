#### 1 通过Mac 的设置 stop mysql





#### 2 跳过权限认证

```shell
// 进入数据库指令文件
cd /usr/local/mysql/bin
// 跳过权限认证
sudo ./mysqld_safe --skip-grant-tables
```





#### 3 免密码进入数据库

##### **新开**一个终端，同时**保持原来那个终端也开着**，在新的终端输入如下指令：

```shell
//  执行mysql指令
/usr/local/mysql/bin/mysql
// 进入名为<mysql>的数据库
use mysql
// 刷新权限
flush privileges;
// 修改密码 但不适用于8.0+的版本
// set password for 'root'@'localhost' = password('新的密码');
// 8.0+版本修改密码
alter user 'root'@'localhost' identified by '新密码';

// 退出mysql
exit
```





#### 4 kill 掉以前mysql 进程，并重新启动mysql； 输入新密码 ok