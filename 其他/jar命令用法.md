#### jar 命令选项：

##### jar命令格式：jar {c t x u f }\[ v m e 0 M i \][-C 目录]文件名...

```shell
jar命令格式：jar {c t x u f }[ v m e 0 M i ][-C 目录]文件名...

其中{ctxu}这四个选项必须选其一。[v f m e 0 M i ]是可选选项，文件名也是必须的。
-c  创建一个jar包
-t  显示jar中的内容列表
-x  解压jar包
-u  添加文件到jar包中
-f  指定jar包的文件名
-v  生成详细的报造，并输出至标准设备
-m  指定manifest.mf文件.(manifest.mf文件中可以对jar包及其中的内容作一些一设置)
-0  产生jar包时不对其中的内容进行压缩处理
-M  不产生所有文件的清单文件(Manifest.mf)。这个参数与忽略掉-m参数的设置
-i    为指定的jar文件创建索引文件
-C  表示转到相应的目录下执行jar命令,相当于cd到那个目录，然后不带-C执行jar命令
```



例子：

```shell
# 将当前目录下的文件打成 xxx.war
jar -cvf xxx.war ./*
```

