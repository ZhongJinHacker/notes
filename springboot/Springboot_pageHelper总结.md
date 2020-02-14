### Springboot上使用pageHelper总结

#### 1. 在pom.xml 中引入依赖

```xml
		<!--pagehelper-->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.5</version>
		</dependency>
```



之后直接在要访问数据库的mapper接口之前使用pageHelper即可，十分简单

```java
    @PostMapping("/allUser")
    public PageInfo<User> allUser() {
        PageHelper.startPage(2,3);
        List<User> users = userMapper.findAllUser();
        PageInfo<User> pageInfo = new PageInfo<>(users);
        return pageInfo;
    }
```

#### 经测试，这里实际上是执行了两条SQL语句（与我的猜测一致）

```java
c.g.m.m.UserMapper.findAllUser_COUNT     : ==>  Preparing: SELECT count(0) FROM user 
c.g.m.m.UserMapper.findAllUser_COUNT     : ==> Parameters: 
 c.g.m.m.UserMapper.findAllUser_COUNT     : <==      Total: 1
 c.g.m.mapper.UserMapper.findAllUser      : ==>  Preparing: SELECT * FROM user LIMIT ?, ? 
c.g.m.mapper.UserMapper.findAllUser      : ==> Parameters: 3(Integer), 3(Integer)
 c.g.m.mapper.UserMapper.findAllUser      : <==      Total: 3
```

执行了一条统计总数（where 真正要执行的sql的条件）的SQL 和 真正的SQL



顺便检查出来了我的一个SQL上理解的错误：

### Limit x, y 指的是从x的下一条数据开始，包含y条数据数据的集合（不包含x）



#### 2. ==执行的mapper接口一旦在上面使用了PageHelper.startPage(x,y); 之后返回的对象其实不再是List对象了，而是一个Page对象(继承于ArrayList)==

```java
class Page<E> extends ArrayList<E> implements Closeable
```

#### Page 对象中保存了两次SQL返回的信息，这样就得到了total的值

PS: 经测试如果mapper接口之上没有使用**PageHelper.startPage(x,y);**则返回的还是ArrayList



#### 3. 要拿到total的值需要用PageInfo做一层封装即可

```java
PageInfo<User> pageInfo = new PageInfo<>(users);
```

