### Springboot 多属性文件配置

**配置文件后缀有两种:** ==.properties==和==.yml==



要完成多属性配置需要自定义==PropertySourcesPlaceholderConfigurer== 这个Bean



**properties配置方法**

```java
    /**
     * 这里必须是static函数
     * 如果不是 application.propertise 将读取不到
     * application.properties 是默认加载的，这里配置自己的properties就好
     * @return
     */
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        // properties 加载方式
        Resource[] resources = {
                //new ClassPathResource("application.properties"),
          			//ClassPathResource针对的是resource目录下的文件
                new ClassPathResource("test.properties")
        };
        configurer.setLocations(resources);
        return configurer;
    }
```

#### 注意这个Bean 的函数必须是static的，否则会加载不到application.properties中的内容

==这里不需要将application.properties也加进来，因为application.properties是默认加进来的，这里只要写其他的属性文件就好了==



**yml配置方法**

```java
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();

        // yml 加载方式
				YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
      	//这里可以传入多个自定义属性文件
				yaml.setResources(new ClassPathResource("test.yml"));
				configurer.setProperties(yaml.getObject());

        configurer.setLocations(resources);
        return configurer;
    }
```



**yml 和properties的配置的注意点是一样的，只是需要==YamlPropertiesFactoryBean==来加载**



#### 2. 如果引用到属性文件中的值

**test.propertise**

```properties
grady.username=jiang
grady.password=1234567
```

**TestConfig.java**

 ```java
@Configuration
// 这里不需要了
//@PropertySource("classpath:test.properties")
public class TestConfig {

    @Value("${grady.username}")
    private String username;

    @Value("${grady.password}")
    private String password;

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}

 ```

#### 这里用@Configuration或@Component都可以获得到值，

==注意：==这里不需要使用==@PropertySource==了，直接用就可以了



#### 3. 在Controller中使用（其他地方也可，这里是举例）

 ```java
public class UserController {

    @Autowired
    private TestConfig testConfig;

    @Autowired
    private SystemConfig systemConfig;

    @PostMapping("/hello")
    public String Hello() {
        String datasourcePassword = systemConfig.getDatasourcePassword();
        return "Hello World" + testConfig.getUsername() + "  " + testConfig.getPassword()
                + "  datasourcePassword= " + datasourcePassword;
    }
 ```



**postMan中的结果**

```json
Hello Worldjiang  1234567  datasourcePassword= Root123#
```



#### 4. 更简洁的写法

```java
@Configuration
public class SystemConfig {

    private static List<Resource> resourceList = new ArrayList<>();

    static {
        resourceList.add(new ClassPathResource("test.properties"));
        resourceList.add(new ClassPathResource("jdbc.properties"));
    }

    @Value("${spring.datasource.password}")
    private String datasourcePassword;

    /**
     * 这里必须是static函数
     * 如果不是 application.propertise 将读取不到
     * application.properties 是默认加载的，这里配置自己的properties就好
     * @return
     */
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        // properties 加载方式
        configurer.setLocations(resourceList.stream().toArray(Resource[]::new));
        return configurer;
    }

    public String getDatasourcePassword() {
        return datasourcePassword;
    }
}
```

