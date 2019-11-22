### git stash

#### 1. git stash save "message"

​	执行存储，并添加备注信息（直接git stash 也可以，但没有备注信息）

#### 2. git stash list

​	查看存储列表

```shell
stash@{0}: On Topic/V2.5: gitignore和mybatis日志
stash@{1}: On Topic/V2.5: test
```



#### 3. git stash show

​	显示第一个存储做了哪些改动，要查看其他的，**后面加**==stash@{$num}==

​	比如**git stash show stash@{1}**

#### 4. git stash show -p

 	显示第一个村粗详细的改动，要查看其他的，****后面加****==stash@{$num}

```shell
diff --git a/.gitignore b/.gitignore
index abee9e6..90c0884 100644
--- a/.gitignore
+++ b/.gitignore
@@ -30,4 +30,5 @@ test/
 logs/
 disconf/

+.DS_Store

diff --git a/idspTransferService-dao/src/main/resources/META-INF/mybatis-config.xml b/idspTransferService-dao/src/main/resources/META-INF/mybatis-config.xml
index 8ca4edd..ed6ee9d 100644
--- a/idspTransferService-dao/src/main/resources/META-INF/mybatis-config.xml
+++ b/idspTransferService-dao/src/main/resources/META-INF/mybatis-config.xml
@@ -6,7 +6,8 @@
     <settings>
         <setting name="cacheEnabled" value="true"/>
         <!-- 打印查询语句 -->
-        <setting name="logImpl" value="LOG4J"/>
+<!--        <setting name="logImpl" value="LOG4J"/>-->
+        <setting name="logImpl" value="STDOUT_LOGGING" />
         <!-- 映射ID到对象 -->
         <setting name="useGeneratedKeys" value="true"/>
     </settings>
(END)
```



#### 5. git stash apply

​    **应用一次个存储，==但不从村粗列表删除==**，

​	**如果要使用其他个，git stash apply stash@{$num}**

#### 6. git stash pop

​	从存储列表中弹出第一个存储（相当于恢复后删除）

​	**如果要应用并删除其他stash，命令：git stash pop stash@{$num} **



#### 7. git stash drop stash@{$num}

​    丢弃stash@{$num}存储，从列表中删除这个存储