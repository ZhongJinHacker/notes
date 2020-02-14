今天基于mybatis 做了一个动态选择查询字段的sql

发现mybatis 有两种传值的方式`${}`和`#{}`

`#{}`会在传入的值两边加入双引号，这样当如想在**表名**或者 **字段名** 上操作时，这里是不行的，

比如我想查询 hello 这个字段 ，我这么写的：

```java
String field = "hello"
  
 ---
 mybatis的xml中
 SELECT #{field} FROM table;
执行的结果会变为：
  SELECT "hello" FROM table;

这样是查不出结果的
  
---
  如果一定要这样操作，需要使用${}
比如
  SELECT ${field} FROM table;
```



PS:  请尽量使用#{}, 因为有双引号，可以防止SQL注入问题

​		使用${}注意保护

#### 默认情况下,使用 `#{}` 格式的语法会导致 MyBatis 创建 `PreparedStatement` 参数占位符并安全地设置参数（就像使用 ? 一样）。 这样做更安全，更迅速，通常也是首选做法.





###不过有时你就是想直接在 SQL 语句中插入一个不转义的字符串,你可以使用`${}`

