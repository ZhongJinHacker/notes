### Springboot 事务

### 1. 打印SQL 日志的两种配置方式

[1]通过配置包的log等级来打印SQL日志，==但这种不会打印出**事务**日志==

```properties
logging.level.com.grady.mybatisdemo.mapper=DEBUG
```

[2] 配置mybatis的logImpl属性，==好处是会输出**事务**日志==

```properties
mybatis.configuration.logImpl=org.apache.ibatis.logging.stdout.StdOutImpl
```



#### PS：==rollbackFor==默认是非受检异常回滚，==配置rollbackFor = Exception.class==改成所有异常回滚



### 2 成功和失败的日志信息

#### [1]. 一次查询成功的日志

代码

```java
    @Transactional(rollbackFor = Exception.class)
    @PostMapping("/allUser")
    public PageInfo<User> allUser() {
        PageHelper.startPage(2,3);
        List<User> users = userMapper.findAllUser();
        PageInfo<User> pageInfo = new PageInfo<>(users);
        return pageInfo;
    }
```



```properties
Creating a new SqlSession
Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7580f2b1]
Cache Hit Ratio [SQL_CACHE]: 0.5
JDBC Connection [HikariProxyConnection@2138610113 wrapping com.mysql.cj.jdbc.ConnectionImpl@55855042] will be managed by Spring
==>  Preparing: SELECT count(0) FROM user_copy 
==> Parameters: 
<==    Columns: count(0)
<==        Row: 5
<==      Total: 1
==>  Preparing: SELECT * FROM user_copy LIMIT ?, ? 
==> Parameters: 3(Integer), 3(Integer)
<==    Columns: user_id, name, password, token, ownapps, role_id
<==        Row: 4, testUser4, 4, a, 2, 4
<==        Row: 5, testUser5, 5, a, 4, 5
<==      Total: 2
Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7580f2b1]
Transaction synchronization committing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7580f2b1]
Transaction synchronization deregistering SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7580f2b1]
Transaction synchronization closing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7580f2b1]
```



#### [2]. 一次抛异常回滚的日志

代码：

```java
    @Transactional(rollbackFor = Exception.class)
    @Override
    public Integer insertUser(User user) throws BusinessException {
        try {
            List<User> users = userMapper.findAllUser();
            Integer result = userMapper.insert(user);
            if (true) {
                int i = 1 / 0; //这里抛RuntimeException异常
            }
            return result;
        } catch (Exception e) {
            e.printStackTrace();
            throw new BusinessException("访问数据库失败", 1001);// 被我catch然后改为受检异常抛出
        }
    }
```

```properties
Creating a new SqlSession
Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@2e775383]
JDBC Connection [HikariProxyConnection@1016081606 wrapping com.mysql.cj.jdbc.ConnectionImpl@3560cd99] will be managed by Spring
==>  Preparing: SELECT * FROM user_copy 
==> Parameters: 
<==    Columns: user_id, name, password, token, ownapps, role_id
<==        Row: 1, testUser1, 1, a, 2, 1
<==        Row: 2, testUser2, 2, a, 2, 2
<==        Row: 3, testUser3, 3, a, 2, 3
<==        Row: 4, testUser4, 4, a, 2, 4
<==        Row: 5, testUser5, 5, a, 4, 5
<==      Total: 5
Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@2e775383]
Fetched SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@2e775383] from current transaction
==>  Preparing: INSERT INTO user_copy ( name, password, token, ownapps, role_id ) VALUES ( ?, ?, ?, ?, ? ) 
==> Parameters: jiang(String), zhongjin(String), abcd(String), o(String), 2(Long)
<==    Updates: 1
Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@2e775383]
Transaction synchronization deregistering SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@2e775383]
Transaction synchronization closing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@2e775383]
```



###ps：此异常从被Transactional注解的方法中抛出后就立即回滚并关闭SQLSession了



###  总结1：SpringBoot使用事务就是这么容易，只要在对应的方法上加上注解就可以了



### 3. 事务冲突问题

**当使用==事务增强==来为一些匹配的方法加上==事务==，如果在这些方法调用其他被@Transactional绑定的方法时**，会抛出事务异常，但对数据不会有影响**

比如：

==事务增强加在了Controller上，controller调用Manager的方法，而Manager方式使用了**@Transactional**注解，这是会抛一下错误（经测试如果加载controller自己上没有问题，在调用链上才有问题）==

抛出的异常是

```properties
org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.processRollback(AbstractPlatformTransactionManager.java:873) ~[spring-tx-5.1.9.RELEASE.jar:5.1.9.RELEASE]
	at org.springframework.transaction.support.AbstractPlatformTransactionManager.commit(AbstractPlatformTransactionManager.java:710) ~[spring-tx-5.1.9.RELEASE.jar:5.1.9.RELEASE]
```



##### [1] 使用事务增强的两种方式

​    **①XML 方式**(然后使用@ImportResource("classpath:transaction.xml")引入)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd">
  <bean id="txManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" ></property>
  </bean>
  <tx:advice id="cftxAdvice" transaction-manager="txManager">
    <tx:attributes>
      <tx:method name="query*" propagation="SUPPORTS" read-only="true" ></tx:method>
      <tx:method name="get*" propagation="SUPPORTS" read-only="true" ></tx:method>
      <tx:method name="select*" propagation="SUPPORTS" read-only="true" ></tx:method>
      <tx:method name="*" propagation="REQUIRED" rollback-for="Exception" ></tx:method>
    </tx:attributes>
  </tx:advice>
   <aop:config>
    <aop:pointcut id="allManagerMethod" expression="execution (* com.grady.mybatisdemo..controller.*.*(..))" />
    <aop:advisor advice-ref="txAdvice" pointcut-ref="allManagerMethod" order="0" />
  </aop:config>
</beans>
```

// read-only="true" 表示这个方法只能读数据库，写会报错

#### 这个增强表示，controller下的所有方法都在范围内，如果满足tx:method 的条件则加上事务



#### ②代码配置方式

```java
@Configuration
public class TransactionAdviceConfig {

    private static final String AOP_POINTCUT_EXPRESSION = "execution (* com.grady.mybatisdemo..controller.*.*(..))";

    @Autowired
    private PlatformTransactionManager transactionManager;

    @Bean(name = "txAdvice")
    public TransactionInterceptor txAdvice() {
        Properties properties = new Properties();
      	// readOnly 表示这个方法只能读数据库，写会报错
        properties.setProperty("get*", "PROPAGATION_REQUIRED,-Exception,readOnly");
        properties.setProperty("add*", "PROPAGATION_REQUIRED,-Exception");
        properties.setProperty("save*", "PROPAGATION_REQUIRED,-Exception");
        properties.setProperty("update*", "PROPAGATION_REQUIRED,-Exception");
        properties.setProperty("delete*", "PROPAGATION_REQUIRED,-Exception");
      	//"PROPAGATION_REQUIRED,  -Exception"这里有空格不要紧
        properties.setProperty("insert*", "PROPAGATION_REQUIRED,-Exception");
        TransactionInterceptor transactionInterceptor = new TransactionInterceptor(transactionManager, properties);
        return transactionInterceptor;
    }

    @Bean
    public Advisor txAdviceAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression(AOP_POINTCUT_EXPRESSION);
        return new DefaultPointcutAdvisor(pointcut, txAdvice());
    }
}
```

**经测试  properties.setProperty("insert*", "PROPAGATION_REQUIRED,-Exception"); =="PROPAGATION_REQUIRED,-Exception"== 每个==关键字的两侧==有==空格==不要紧，加载前会自动trim**

#### 代码的方式要更加优雅，可以利用到编译器在编译期做检查，XML只有运行时才能知道是对是错，代码的方式相对会更好一些



### 总结： 

1. 建议不要使用事务增强，因为你还得去记哪些方法名会被加事务，哪些不会，也会影响到你对方法的命名，一个表意的函数名是非常重要的，我觉得这种方式是得不偿失的。
2. 建议直接使用@Transactional 来为方法加事务，这样一眼就可以知道当前方法有没有事务。
3. 最好不要用了==事务增强==，又用@Transactional，因为如果两个方法加在了同一个方法链上，一旦抛异常回滚是会报UnexpectedRollbackException异常错误的

4. 如果在调用链很后的地方抛出的异常，到被事务增强的函数时，此时一定要接着抛出，否则不会回滚，因为内部消化的异常等于正常操作，不会回滚