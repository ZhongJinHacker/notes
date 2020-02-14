#### 1. 尽量用XML 集成，这也的Dubbo官方推荐的集成方式

 自己在使用注解集成过程中发现有坑：==Springmvc包扫描和dubbo包扫描冲突，导致消费端一直拿不到代理对象（null），非常蛋疼，所以猜测可能还有其他坑==



搭建环境主要是几个pom文件非常重要

项目结构：

```shell
dubbo-demo
	|- common
				|- pom.xml
  |- consumer
  			|- pom.xml
  |- provider
  			|- pom.xml
  |- pom.xml
```



dubbo-demo 的pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.grady</groupId>
  <artifactId>dubbo-demo</artifactId>
  <packaging>pom</packaging>
  <version>1.0-SNAPSHOT</version>

  <modules>
    <module>consumer</module>
    <module>provider</module>
    <module>common</module>
  </modules>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
    <dubbo.version>2.5.5</dubbo.version>
    <zkclient.version>0.10</zkclient.version>
    <lombok.version>1.18.2</lombok.version>
    <spring-boot.version>2.1.3.RELEASE</spring-boot.version>
    <spring-boot-starter-dubbo.version>1.0.0</spring-boot-starter-dubbo.version>

    <slf4j.version>1.7.25</slf4j.version>
    <curator.version>1.1.10</curator.version>
  </properties>


  <dependencyManagement>
    <dependencies>
      <!-- Springboot依赖 -->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>${spring-boot.version}</version>
      </dependency>

      <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <scope>provided</scope>
      </dependency>

      <!-- Dubbo依赖 -->
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
        <exclusions>
          <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
          </exclusion>
        </exclusions>
      </dependency>

      <!--zookeeper的客户端依赖-->
      <dependency>
        <groupId>com.101tec</groupId>
        <artifactId>zkclient</artifactId>
        <version>${zkclient.version}</version>
      </dependency>

    </dependencies>
  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
          <encoding>UTF-8</encoding>
        </configuration>
      </plugin>

      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-resources-plugin</artifactId>
        <configuration>
          <encoding>UTF-8</encoding>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

主要是作为父pom用



然后的common

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <artifactId>dubbo-demo</artifactId>
    <groupId>com.grady</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>

  <artifactId>common</artifactId>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <scope>provided</scope>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

common 主要是作为公共的jar来引用



再是provider

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <artifactId>dubbo-demo</artifactId>
    <groupId>com.grady</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>

  <artifactId>provider</artifactId>

  <dependencies>
    <dependency>
      <groupId>com.grady</groupId>
      <artifactId>common</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <exclusions>
        <exclusion>
          <groupId>log4j</groupId>
          <artifactId>log4j</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <!--zookeeper的客户端依赖-->
    <dependency>
      <groupId>com.101tec</groupId>
      <artifactId>zkclient</artifactId>
      <version>${zkclient.version}</version>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

关键是provider的resource目录下的内容

Spring-dubbo-provider.xml

使用==@ImportResource({"classpath:spring-dubbo-provider.xml"})==进行配置文件加载

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

  <dubbo:application name="provider-demo"/>
  <!-- 注册中心的ip地址 -->
  <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
  <!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->
  <dubbo:annotation package="com.grady.service.impl"/>
  <!-- use dubbo protocol to export service on port 20880 -->
  <dubbo:protocol name="dubbo" port="20880"/>

  <bean id="userService" class="com.grady.service.impl.UserServiceImpl"/>
  <dubbo:service interface="com.grady.service.UserService" ref="userService"/>
</beans>
```





再看consumer的pom.xml

 ```shell
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <artifactId>dubbo-demo</artifactId>
    <groupId>com.grady</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  <modelVersion>4.0.0</modelVersion>

  <artifactId>consumer</artifactId>

  <dependencies>
    <dependency>
      <groupId>com.grady</groupId>
      <artifactId>common</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <exclusions>
        <exclusion>
          <groupId>log4j</groupId>
          <artifactId>log4j</artifactId>
        </exclusion>
      </exclusions>
    </dependency>

    <!--zookeeper的客户端依赖-->
    <dependency>
      <groupId>com.101tec</groupId>
      <artifactId>zkclient</artifactId>
      <version>${zkclient.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
 ```



consumer 的resource目录下的内容

spring-dubbo-consumer.xml

使用==@ImportResource({"classpath:spring-dubbo-consumer.xml"})==进行加载

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

  <dubbo:application name="consumer-demo"/>
  <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
  <dubbo:protocol name="dubbo" port="20880"/>
  <!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->
  <dubbo:annotation package="com.grady.controller"/>

  <dubbo:reference id="userService" check="false" interface="com.grady.service.UserService"/>
</beans>
```

---

将consumer 和provider 启动后，查看Zookeeper里的内容

先只启动provider

```shell
[zk: localhost:2181(CONNECTED) 18] ls /dubbo/com.grady.service.UserService/consumers
[]
```

可以看到consumers没有内容

```shell
[zk: localhost:2181(CONNECTED) 19] ls /dubbo/com.grady.service.UserService/providers
[dubbo%3A%2F%2F100.118.67.11%3A20880%2Fcom.grady.service.UserService%3Fanyhost%3Dtrue%26application%3Dprovider-demo%26dubbo%3D2.5.5%26generic%3Dfalse%26interface%3Dcom.grady.service.UserService%26methods%3DfindUser%26pid%3D51651%26side%3Dprovider%26timestamp%3D1566208057732]
```

可以看到providers中有提供者的信息（比如ip，提供的借口等）



#### 再启动consumer

```shell
zk: localhost:2181(CONNECTED) 20] ls /dubbo/com.grady.service.UserService/consumers
[consumer%3A%2F%2F100.118.67.11%2Fcom.grady.service.UserService%3Fapplication%3Dconsumer-demo%26category%3Dconsumers%26check%3Dfalse%26dubbo%3D2.5.5%26interface%3Dcom.grady.service.UserService%26methods%3DfindUser%26pid%3D51946%26side%3Dconsumer%26timestamp%3D1566208217754]
```

consumer启动成功后可以看到Zookeeper中有对应consumer的内容