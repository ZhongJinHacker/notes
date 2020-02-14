#### tail 命令用法

功能从尾部显示文件若干行

语法：

```shell
 tail  [ +/- num ][参数] 文件名
```

使用tail命令的-f选项可以方便的查阅正在改变的日志文件，**tail -f filename**会把filename里最尾部的内容显示在屏幕上，<u>并且不断刷新</u>，使你看到最新的文件内容

例子：

1. 显示最后20行

   ```
   tail -20 test.txt
   ```

2. 不断监听文件追加的新内容（默认10行）

   等同于--follow=descriptor，根据文件描述符进行追踪，当文件改名或被删除，追踪停止

   ```shell
   tail -f test.txt
   
   ```

