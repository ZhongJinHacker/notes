dev分支

```shell
* da349ef (dev) e
* 75350bc d
* 63cbbb8 c
* c6509a5 b
* 13405af a
```



文件可能会发生冲突，需要解决一下

```shell
aaaaaaaaa


bbbbbbbbb


ccccccccc

<<<<<<< HEAD
fffffffff


ggggggggg
=======

ddddddddd

eeeeeeeee
>>>>>>> dev
```



最后结果

```shell
*   85ef130 (HEAD -> master) Merge branch 'dev'
|\
| * da349ef (dev) e
| * 75350bc d
* | b553464 g
* | 6d56d3d f
|/
* 63cbbb8 c
* c6509a5 b
* 13405af a
```

可以看出，merge 有保留dev分支的痕迹，相当于创建了一个==85ef130== commit 并同时指向了==da349ef (dev) e==和

==b553464 g==

---

### 再看rebase是如何处理的

Master 分支

```shell
* 2a4e835 (HEAD -> master) d
* 706b742 c
* 4807917 b
* 59a582d a
```



dev 分支

```shell
* 0a6ecdd (HEAD -> dev) f
* 42b6702 e
* 706b742 c
* 4807917 b
* 59a582d a
```



#### 使用rebase合并分支

在master 上执行

```shell
git rebase dev
```

（==着重理解这句话==）相当于将master分支的代码向dev分支合并,在将master的指针指向dev上新创建的commit



可能会有冲突，解一下

```shell
aaaaaa

bbbbbb

cccccc

<<<<<<< HEAD //这里是dev 的内容
eeeeee

ffffff
=======
dddddd
>>>>>>> d // 这里是master的内容
```

#### 冲突时，冲突部分与merge不同，相当于将master的差异代码往dev合并，然后产生了一个新的commit，之后将master变基到这个新的commit-id 上 



解决冲突后执行 **git add <conflicted_files>** ，然后执行==git rebase --continue== 

rebase操作是持续执行的，如果还有冲突，继续执行上述操作（**好处是：也方便一个冲突一个冲突的解决**）



rebase 合并成功后（有的也翻译为变基），在master上查看git日志

```shell
* ff1a394 (HEAD -> master) d //这里是新创建的一个commitid，commit注释为d master变基到此
* 0a6ecdd (dev) f
* 42b6702 e 
* 706b742 c
* 4807917 b
* 59a582d a
```



### 可以看出，在master分支执行git rebase dev 是将当前分支的代码往dev合入，并最后将master的指针指向到在dev链上新创建的commit，完成合入（变基）

