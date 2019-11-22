### shadow 文件权限

```shell
$ll shadow                                                                                                                                         
---------- 1 root root 1131 Aug  6 12:04 shadow 
```

可以看出只有root可以操作它，普通用户执行passwd，内部也是先隐式切到root，改完再回到普通用户

### shadow文件内容

```shell
# head -3 /etc/shadow

root:$6$kcgcu794R0VP3fDL$aYN8XUbtWvZ4QQtT2xVW.N2CgE3YLPdtnprAAtKZUgNdq8itUJEN6NoYQDarLUevcDCWrxMVId8b18ujwST1b0::0:99999:7:::
bin:*:17632:0:99999:7:::
daemon:*:17632:0:99999:7:::	
```

#### 内容分析： 以“:”分隔

[1] 第一个字段：用户名

[2] 第二个字段：==**加密后的密码**==

[3] 第三个字段：最后一次修改密码的时间

[4] 第四个字段：密码需要重新设置的天数（99999表示不用重新设置）

[5] 第五个字段：密码变更前提前几个警告 （比如：7）

[6] 第六个字段：账号时效日期

[7] 第七个字段：账号取消日期

[8] 第八个字段：保留条目，未使用



### 第二个字段，即加密后的密码的解释

shadow文件的密码部分由三个部分组成，由`'$'`分割。

比如：

`$6$kcgcu794R0VP3fDL$aYN8XUbtWvZ4QQtT2xVW.N2CgE3YLPdtnprAAtKZUgNdq8itUJEN6NoYQDarLUevcDCWrxMVId8b18ujwST1b0`

被分为了三部分：

==第一部分==`6`为：**加密方式**

```
目前加密方式有6种，最常见的只有3种： 
1:MD5加密，密文长度22 
5:SHA-256加密，密文长度43 
6:SHA-512加密，密文长度86 
```



==第二部分==`kcgcu794R0VP3fDL`为：**salt值**



==第三部分==

`aYN8XUbtWvZ4QQtT2xVW.N2CgE3YLPdtnprAAtKZUgNdq8itUJEN6NoYQDarLUevcDCWrxMVId8b18ujwST1b0`

为：**加密后的密码串**



### 最后 shadow文件非常重要，也只有root可以查看，如果被人查看或者说窃取到了shadow文件，可以使用暴力破解的方式逆推密码。