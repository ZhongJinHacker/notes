### git revert

#### git revert 是一种创建一次==新==的commit 来==回退==**某次或某几次commit**的一种方式



命令

```shell
// 创建一个新的commit，这个commit会删除（下面）commit-id的内容，但会在log中保留这个commit-id
git revert commit-id

// 批量回退 这个是前开后闭, 只revert了...和commit-id-end, commit-id-start没有撤销
git revert commit-id-start...commit-id-end

// 加上^ 就是前闭后闭，都撤销了
git revert commit-id-start^...commit-id-end
```



#### ps： 没有**git revert 文件**的操作，IDEA的撤销文件修改的操作（revert）用的不是**git revert**



例子：

原始log

```shell
31f6a64 (HEAD -> master) 提交 3
98d937c 提交 2
af02354 提交 1
```

目录下的内容为:

```shell
➜  revert_test git:(master) ls
 1.txt
 2.txt
 3.txt
```



现在想删除 ==98d937c== 的提交

执行

```shell
git revert 98d937c
```



再看目录下的内容为：

```shell
➜  revert_test git:(master) ls
 1.txt
 3.txt
```

说明确实去掉了==98d937c== 的提交内容（**即2.txt 不在了**）

再看日志：

```shell
d1ea1e9 (HEAD -> master) Revert "提交 2"
31f6a64 提交 3
98d937c 提交 2
af02354 提交 1
```

发现，其实是通过一次新的提交来删除==98d937c== 的提交内容的(这里可以看出之前的commit-id依然在)





#### 再看批量回退

日志记录

```shell
a74b9bb (HEAD -> master) 提交 6
e3bbc59 提交 5
223171c 提交 4
d1ea1e9 Revert "提交 2"
31f6a64 提交 3
98d937c 提交 2
af02354 提交 1
```

现在回退 提交4 和提交5

执行

```shell
git revert 223171c^...e3bbc59
```

看日志

```shell
af51f68 (HEAD -> master) Revert "提交 4"
1a5d0c0 Revert "提交 5"
a74b9bb 提交 6
e3bbc59 提交 5
223171c 提交 4
d1ea1e9 Revert "提交 2"
31f6a64 提交 3
98d937c 提交 2
af02354 提交 1
```

产生了两次revert提交commit