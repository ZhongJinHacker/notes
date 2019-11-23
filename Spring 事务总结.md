# Spring 事务总结

### rollbackFor 设为 Exception.class场景下

1. 如果在函数内部catch住异常消费掉，没有再抛出的话，不会回滚
2. 如果catch住 然后原封不动抛出，会回滚
3.  如果catch住，然后改造成其他异常抛出，会回滚

4. 如果是内层函数抛出，外层带事务的函数未抛出，也不会回滚
5. 如果外层带事务函数catch住再抛出，会回滚
6. 事务函数调用本类的public带有事务的函数，第二个函数不会带有事务，相当于一个普通函数，除非是调用其他类的事务函数
7. 如果是***@Transactional(propagation= Propagation.==REQUIRED==, rollbackFor = Exception.class)*** 调用的对象函数也是==REQUIRED==，则被调用函数成功抛出异常，即使外部函数catch住异常不抛，也会成功回滚，因为是同一个事务，aop在被调函数的增强中已经处理了回滚逻辑



### 两种配置方式

### 1. 注解配置 ==@EnableTransactionManagement==，之后就可以@Transactional了

### 2.XML配置，resource下创建XML文件==spring-tx.xml==，之后==@ImportResource(locations = {"classpath:spring-tx.xml"})==，

XML配置还要引入jar： 

```xml
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>1.7.4</version>
		</dependency>
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 		  xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 配置Spring的事务管理器 -->
    <bean id="transactionManager"   	class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 拦截器方式配置事务 -->
    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" rollback-for="Exception" />
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="transactionPointcut" expression="within(com.grady.demotransaction..impl.*Impl) &amp;&amp; execution(* *(..))" />
        <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" />
    </aop:config>
</beans>
```



***aop:pointcut 的 expression 一定要配置正确***， 之后就tx:method 中匹配的函数不需要@transactional了



### 3、 XML 配置注解型的事务 

```xml
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

    <!-- 配置Spring的事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- XML配置中开启事务，之后可用注解@Transactional -->
    <tx:annotation-driven transaction-manager="transactionManager" />
</beans>
```



 如果springboot版本大于2.1  ,properties 中要加上这个

```yml
spring:
  main:
    allow-bean-definition-overriding: true
```

### 经测试，同时XML配置注解版和拦截器版时，也是可以的，但建议只用其中之一

```xml
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

    <!-- 配置Spring的事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- XML配置中开启事务，之后可用注解@Transactional -->
    <tx:annotation-driven transaction-manager="transactionManager" />

    <!-- 拦截器方式配置事务 -->
    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="test*" propagation="REQUIRED" rollback-for="Exception" />
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="transactionPointcut" expression="within(com.grady.demotransaction..impl.*Impl) &amp;&amp; execution(* *(..))" />
        <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice" />
    </aop:config>

</beans>
```



最后建议还是使用注解的方式，更加明白直接，因为拦截器有可能由漏网之鱼，问题难以排查