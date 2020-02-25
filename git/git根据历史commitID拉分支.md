##### 1. git log -g  查看已commit的信息

##### 2.  根据commit信息找到对应的commitID

##### 3. 执行一下命令来创建新的分支

```shell
### 1. 方法一：创建一个基于commitId的分支，但不切过去
git branch new_branch_name commitId

### 2. 方法二：创建基于commitId的分支，并切过去
git checkout -b new_branch_name commitId
```

