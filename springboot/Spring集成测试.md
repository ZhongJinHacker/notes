### Spring 集成测试

#### 需要再类的头部加入

 ```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:META-INF/spring/*.xml"})
 ```

==@ContextConfiguration==也可以改为一下具体的写法， 即指向具体的文件

```java
@ContextConfiguration(locations =
        {
                "classpath:META-INF/spring/spring-config-service.xml",
                "classpath:META-INF/spring/spring-config-datasource.xml",
                "classpath:META-INF/spring/spring-context-quartz.xml",
                "classpath:META-INF/spring/spring-context-task.xml",
                "classpath:META-INF/spring/spring-redis-config.xml",
                "classpath:META-INF/spring/spring-dubbo-service.xml"
        })
```

```java
"classpath*:META-INF/spring/*.xml"
```

### classpath后有通配符==*==时，表示后面的路径可以使用通配符；如果classpath后不追加==*==时，后面必须写具体路径



## 最后,容易忽略的点

```xml
src/main/java：
里面的java文件只能直接加载src/main/resources下的资源，不能直接加载src/test/resources下的资源；
src/test/java: 
里面的java文件既能加载src/test/resources下的资源，又能加载src/main/resources下的资源，当两个resources下都有要加载的同名资源时候，优先选择src/test/java下的资源；
```

也就是说，我们做单元测试时，读取的classpath路径是main和test下的总和的文件；且同名时test下的优先

