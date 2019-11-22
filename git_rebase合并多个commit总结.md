###

git rebase 合并多个commit 方法



> 在开发过程中，有时一个任务会分几次commit提交，这样可能对于有些分支要cherry pick时会比较麻烦，这是我们可以通过git rebase 将几个commit合并为一个commit，再推送到远端



### git rebase -i

这里指的是通过交互的手段执行git rebase， 也是合并commits 的好方法





### 例子

假设当前git日志文件内容是这样

```shell
➜  test git:(master) git log --oneline
bb44232 (HEAD -> master) 增加第f行
5d2e760 增加第b行,增加第c行
f443e25 增加第a行,增加第行
48d5c58 增加第6行,增加第7行,增加第8行
c4b45e6 增加第五行
0469b01 增加第四行,修改commit
cd80e8f 22222
295e1a6 11111
a772b51 1. 增加testgit文件
```

**我想合并这几个commit**

```shell
bb44232 (HEAD -> master) 增加第f行
5d2e760 增加第b行,增加第c行
f443e25 增加第a行,增加第行
48d5c58 增加第6行,增加第7行,增加第8行
```

#### 1. 要先找出==48d5c58 增加第6行,增加第7行,增加第8行==之前的commit-id ==c4b45e6==

执行以下命令

```shell
git rebase -i c4b45e6
```

会进入vim 并显示以下内容

```shell
pick 48d5c58 增加第6行,增加第7行,增加第8行
pick f443e25 增加第a行,增加第行
pick 5d2e760 增加第b行,增加第c行
pick bb44232 增加第f行

# Rebase c4b45e6..bb44232 onto c4b45e6 (4 commands)
#
# Commands:
# p, pick = use commit 
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

命令参数解释：

```shell
pick：保留该commit（缩写:p）

reword：保留该commit，但我需要修改该commit的注释（缩写:r）

edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）

squash：将该commit和前一个commit合并（缩写:s）

fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）

exec：执行shell命令（缩写:x）

drop：我要丢弃该commit（缩写:d）
```



将下面几行的pick 换成==s== 即可，代表将下面几个合并到最近的（即==pick 48d5c58==）commit上，并生成一个新的commit（包括==48d5c58==就都没有了）

```shell
pick 48d5c58 增加第6行,增加第7行,增加第8行
s f443e25 增加第a行,增加第行
s 5d2e760 增加第b行,增加第c行
s bb44232 增加第f行

# Rebase c4b45e6..bb44232 onto c4b45e6 (4 commands)
# ...
```

之后**可能冲突**（跨commit rebase时可能会冲突），就解决冲突

然后执行==git add== 和 ==git rebase --continue==



最后会弹出vim，让你填写这个最后合并的commit的注释信息

填写完 wq，就ok了

执行成功信息

```shell
➜  test git:(master) git rebase -i c4b45e6
[detached HEAD 7d9155b] 增加第6行,增加第7行,增加第8行;增加第a行,增加第行;增加第b行,增加第c行
 Date: Tue Aug 13 17:46:37 2019 +0800
 2 files changed, 17278 insertions(+)
 create mode 100644 package-lock.json
Successfully rebased and updated refs/heads/master.
```

 查看git 日志

```shell
7d9155b (HEAD -> master) 增加第6行,增加第7行,增加第8行;增加第a行,增加第行;增加第b行,增加第c行
c4b45e6 增加第五行
0469b01 增加第四行,修改commit
cd80e8f 22222
295e1a6 11111
a772b51 1. 增加testgit文件
42c86e2 Initial commit from Create React App
```

可以看到c4b45e6之上的commit都合成了一个commit了（即7d9155b，一个新的commitid）

