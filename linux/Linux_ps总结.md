#### ps命令用于监测进程的工作情况。进程是一直处于动态变化中，而ps命令所显示的进程工作状态时瞬间的

使用方式：

```
ps [options]
```

常用参数

```shell
-A 显示所有进程
-a 显示现行终端机下的所有进程，包括其他用户的进程
-u 显示用户的UID
-x：显示没有控制终端的进程，同时显示各个命令的具体路径
e：命令之后显示环境
-f：全部列出，通用和其他选项联用。
-au：显示较为详细的进程信息
-aux：即显示所有进程又显示详细信息
```



常用用法：

```shell
ps a 显示现在终端下的所有程序，包含其他用户的程序。
ps c 列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示。
ps e 列出程序时，显示每个程序所使用的环境变量。
ps f 用ASCII字符显树状结构，表达程序间的相互关系。
ps s 采用程序信号的格式显示程序状况。
ps S 列出程序时，包含已中断的子程序资料。
ps u 以用户为主的格式来显示程序状况。
ps x 显示所有程序，不以终端机来区分。

ps aux
ps ef
```





```shell
~ ps aux
USER               PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
01376019         23833   9.3  3.1  4846924 260048   ??  R     3:05下午   0:09.28 /Applications/iTerm.app/Contents/MacOS/iTerm2
_sophos            507   4.4  1.3  4893444 105864   ??  Us   五02下午  71:01.83 SophosScanD
_windowserver      233   3.3  0.6  7881712  52688   ??  Ss   五02下午  35:45.59 /System/Library/PrivateFrameworks/SkyLight.framework/Resources/WindowServer -daemon
01376019           825   2.2  0.9  5131664  76200   ??  S    五02下午   1:22.03 /System/Library/CoreServices/Finder.app/Contents/MacOS/Finder
01376019          5302   1.5  0.4  4524216  30204   ??  Rs   11:35上午   0:02.81 /System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/DocumentPopoverViewService.xpc/Contents/MacOS/DocumentPopoverViewService
root             24041   0.6  0.0  4279936   1144 s001  R+    3:07下午   0:00.00 ps aux
01376019         23839   0.5  0.0  4296984   3856 s001  S     3:05下午   0:00.26 -zsh
```

字段说明：



PID： 用户进程ID

%CPU: 进程的cpu占用率

%MEM： 进程的内存占用率

VSZ： 该进程使用掉的虚拟内存量 (Kbytes)

RSS ：该进程占用的固定的内存量 (Kbytes)

TTY（TT） ：该进程是在那个终端机上面运作，若与终端机无关，则显示 ?

STAT：该程序目前的状态