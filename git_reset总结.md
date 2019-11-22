### git reset

#### git 的重置操作

#### 有三种模式：==hard==、==mixed（默认）==、==soft==



### 1. hard 用法

==hard==会重置**stage区**和**工作区**，和移动代码库上HEAD 和branch的指针所指向的位置，所有的都没了（干净了），如果工作区或stage区有修改，则全部舍弃了

```shell
//重置到与代码库HEAD所指向的commit处（一般都是最新的commit）
git reset --hard HEAD

//重置到与代码库HEAD的上一个commit处（即第二个，从上往下数）
git reset --hard HEAD^

//重置到与代码库HEAD的上两个commit处（即第三个，从上往下数）
git reset --hard HEAD^^

//重置到与代码库HEAD的上N个commit处
git reset --hard HEAD~N
```

### ps：实际上还是有的，如果你记得之前的commit-id ，还可以checkout过去（checkout游离）

**也就是说hard在代码库的操作是移动了head branch 所指向的位置，而不是删除了commit**



### 2. soft 用法

==soft== **不会重置**stage区和工作区，并将由于移动HEAD 和 Branch 带来的==差异放进stage区==

```shell
//重置到与代码库HEAD所指向的commit处（一般都是最新的commit）
git reset --soft HEAD

//重置到与代码库HEAD的上一个commit处（即第二个，从上往下数）
git reset --soft HEAD^

//重置到与代码库HEAD的上两个commit处（即第三个，从上往下数）
git reset --soft HEAD^^

//重置到与代码库HEAD的上N个commit处
git reset --soft HEAD~N
```

比如

当前暂存区和工作区的内容为

```shell
➜  reset_test git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   d.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   c.txt
```

执行

```shell
➜  reset_test git:(master) ✗ git reset --soft  HEAD^
```

后，暂存区和工作区的内容为

```shell
➜  reset_test git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   c.txt
	new file:   d.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   c.txt
```

可以看到上一个commit的内容**回到了暂存区**内，==这样就相当于可以修改这个commit的内容了==



### 3. mixed (即默认) 用法

**不跟参数，本身就是使用mixed**

比如：

```shell
➜  reset_test git:(master) ✗ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   d.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   c.txt
```

git log 为

```shell
9a4ab25 (HEAD -> master) commit c
72b36c9 commit b
a3ce6eb commit a
9b4b913 第一次提交
```

现在回滚到 commit b

```shell
git reset HEAD^
```

可以看到日志变为

```shell
72b36c9 (HEAD -> master) commit b
a3ce6eb commit a
9b4b913 第一次提交
```

在 git status 看一下

 ```shell
➜  reset_test git:(master) ✗ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	c.txt
	d.txt
 ```

将代码库 暂存区 的修改及工作区的修改一并打包放入了暂存区；==暂存区==被清空



### reset 模式总结

--hard : 代码库代码回退到某个commit，==暂存区和工作区==内容==丢弃==

--soft： 代码库代码回退到某个commit，回退的修改和原暂存区的修改一起放入==暂存区==，**工作区**修改==保留==

--mixed： 代码库代码回退到某个commit，==回退的修改==和==原暂存区的修改==加上==工作区的修改==一并**放入**==工作区==



**由此可见，mixed作为默认是很科学的**





# reset总结

### reset 的本质：移动 HEAD 以及它所指向的 branch

实质上，**reset** 这个指令虽然可以用来撤销 **commit** ，但它的实质行为并不是撤销，而是移动 **HEAD** ，并且「捎带」上 **HEAD** 所指向的 **branch**（如果有的话）。也就是说，**reset** 这个指令的行为其实和它的字面意思 "**reset**"（重置）十分相符：它是用来重置 **HEAD** 以及它所指向的 **branch** 的位置的