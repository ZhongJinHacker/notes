### 在resource下写两个文件

#### 比如：

```shell
resources
- bootstrap.properties
- bootstrap-dev.properties
```

#### bootstrap.properties 的内容

```properties
spring.application.name=hello-dubbo-consumer-config
spring.cloud.nacos.config.server-addr=172.16.5.134:8848
spring.cloud.nacos.config.file-extension=yaml
```

#### bootstrap-dev.properties 的内容

```shell
spring.application.name=hello-dubbo-consumer-config
spring.cloud.nacos.config.server-addr=172.16.5.134:8848
spring.cloud.nacos.config.file-extension=yaml
```



#### 当运行加参数--spring.profiles.active 及对应的配置值时，比如

```shell
java -jar app.jar --spring.profiles.active=prod
```

### 就会读取nacos上的 ==hello-dubbo-consumer-config-prod.yaml==文件



```shell
java -jar hello-apache-dubbo-consumer-0.0.1-SNAPSHOT.jar 
```

