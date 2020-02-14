当由于修改了Git 的密码导致 `pull` 等操作报错时，比如报以下错误：

```fatal: Authentication failed for 'http://xxxxxxxxxxxxxxxxxx.git/'```



可以使用以下命令重置密码

 ```shell
git config --system --unset credential.helper
 ```

再次执行git 操作时就可以输入新的密码了，并会重新再本地得到保存