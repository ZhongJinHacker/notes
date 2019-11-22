### git cherry-pick



#### cherry-pick : 精心挑选，挑选一个我们需要的 commit 进行操作。它可以用于将在==其他分支==上的 commit 移植到==当前的分支==。



#### 用法：

```shell
// 复制commit-id 对应的修改并创建新的commit-id合并到当前分支
git cherry-pick <commit-id>

// 与上相同，并保留原提交的作者的信息
git cherry-pick -x <commit_id>
```



#### 批量挑选：

```shell
// 范围就是 start-commit-id 到 end-commit-id 之间所有的 commit， (左开，右闭] 的区间，也就是说，它将不会包含 start-commit-id 的 commit
git cherry_pick <start-commit-id>…<end-commit-id>


//如果想要包含 start-commit-id 的话，就需要使用 ^ 标记一下，就会变成一个 [左闭，右闭] 的区间
git cherry-pick <start-commit-id>^...<end-commit-id>
```



例子：

```shell
git cherry-pick 371c2…971209 // (2,5]
git cherry-pick 371c2^…971209 // [2,5]
```

