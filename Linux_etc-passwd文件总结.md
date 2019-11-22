#### 文件内容

 ```shell
##
# User Database
#
# Note that this file is consulted directly only when the system is running
# in single-user mode.  At other times this information is provided by
# Open Directory.
#
# See the opendirectoryd(8) man page for additional information about
# Open Directory.
##
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
_uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
_taskgated:*:13:13:Task Gate Daemon:/var/empty:/usr/bin/false
_networkd:*:24:24:Network Services:/var/networkd:/usr/bin/false
_installassistant:*:25:25:Install Assistant:/var/empty:/usr/bin/false
_lp:*:26:26:Printing Services:/var/spool/cups:/usr/bin/false
_postfix:*:27:27:Postfix Mail Server:/var/spool/postfix:/usr/bin/false
_scsd:*:31:31:Service Configuration Service:/var/empty:/usr/bin/false
//省略...
 ```

#### 内容分析： 以“:”分隔

[1] 第一个字段： 用户名称

[2] 第二个字段：x或* 表示密码标志；真正的密码在shadow文件中

[3] 第三个字段：用户ID ，即UID

​								0：超级用户，root，权限最大

​								1-499：系统级用户，系统级用户是系统用来启动相关服务和命令的，不能用来登录系统，而且

​											 不能删除，删除伪用户会造成一些命令不能使用。

​								500-65535：普通用户。Linux内核2.6以后是可以支持232个用户，基本上是不用担心用户不够																			

[4] 第四个字段：用户组ID，即GID

[5] 第五个字段：用户说明信息（==几乎没啥价值，相当于注释==）

[6] 第六个字段：工作目录

​								普通用户：/home/用户名/

​                                超级用户：/root/

[7] 第七个字段：登录之后用的 shell