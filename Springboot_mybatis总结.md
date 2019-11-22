### mybatis 总结



属性配置

#### 1. mybatis.configuration.mapUnderscoreToCamelCase=true

==mapUnderscoreToCamelCase==用于映射表中的字段与==model==中成员的映射关系

映射关系为：表中字段去掉=="_"==,并转为驼峰

比如：

```java
//映射转换方式
role_id => roleId
```

如果配置为**false**，则只有一模一样的字段才可以映射到





### 2. mybatis.mapper-locations

**用于设置mapper.xml的映射目录**

比如

```properties
mybatis.mapper-locations=classpath:/mybatis/mapper/*.xml
```

**这个路径必须写正确，之前写错误有报错以下错误，且还是在运行期发生与数据库访问时报错，无法再编译期检查出来，要格外小心**

```text
2019-08-28 13:52:33.769 ERROR 26395 --- [nio-8080-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.grady.mybatisdemo.mapper.UserMapper.findAllUser] with root cause

org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.grady.mybatisdemo.mapper.UserMapper.findAllUser
	at org.apache.ibatis.binding.MapperMethod$SqlCommand.<init>(MapperMethod.java:235) ~[mybatis-3.5.2.jar:3.5.2]
	at org.apache.ibatis.binding.MapperMethod.<init>(MapperMethod.java:53) ~[mybatis-3.5.2.jar:3.5.2]
	at org.apache.ibatis.binding.MapperProxy.lambda$cachedMapperMethod$0(MapperProxy.java:61) ~[mybatis-3.5.2.jar:3.5.2]
	at java.util.concurrent.ConcurrentHashMap.computeIfAbsent(ConcurrentHashMap.java:1660) ~[na:1.8.0_191]
	at org.apache.ibatis.binding.MapperProxy.cachedMapperMethod(MapperProxy.java:61) ~[mybatis-3.5.2.jar:3.5.2]
```



#### 3. 一定要加上@MapperScan("com.grady.mybatisdemo.mapper")注解

==mapperScan中需填入mapper.java所在的包的路径==，方便mybatis生成增强对象

**经测试@mapperScan可以加载使用@Bean @Configuration @Component @Service 注解的类上，也是可以生效的**

==但建议还是加载Application的类上，这样更加一目了然==



#### 4. mybatis.typeAliasesPackage=com.grady.mybatisdemo.model

**mybatis.typeAliasesPackage**用于指定包的别名映射关系，==之后在xml中就直接使用类名就可以了==

比如：

```xml
  <select id="findAllUser" resultType="User">
    SELECT
      *
    FROM
      user
  </select>
```



##### PS：这个属性虽好，但建议谨慎使用，因为它固定了model类生成的目录，而大型项目往往进行模块划分，不能保证所有的model在同一个目录下，而且还需要去属性文件中去寻找目录的位置，相对会有些繁琐（==IDE也不能通过左击直接进入model类中）



#### 5. 开启Mybatis SQL 日志

```properties
logging.level.com.grady.mybatisdemo.mapper=DEBUG
```

