# SpringBoot多重属性文件配置方案笔记



1. 需要重写==**PropertyPlaceholderConfigurer**==

2. 同时要忽略**==DataSourceAutoConfiguration==**

   ```java
   @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
   //@EnableTransactionManagement 2. 尝试xml配置事务
   @ImportResource(locations = {"classpath:spring-tx.xml"})
   public class DemoTransactionApplication {
   
   	public static void main(String[] args) {
   		SpringApplication.run(DemoTransactionApplication.class, args);
   	}
   
   }
   ```

3. 由于忽略了**DataSourceAutoConfiguration**，所以要配置datasource的Bean，和sqlSessionFactory 的Bean

   ```java
   @Configuration
   @MapperScan(basePackages = {MyBatisConfig.MAPPER_PACKAGE}, sqlSessionFactoryRef = MyBatisConfig.SESSIONFACTORY_NAME)
   public class MyBatisConfig {
   
       /**SqlSessionFactory名称.*/
       public final static String SESSIONFACTORY_NAME = "sqlSessionFactory";
       /**mapper包路径，必须与其他SqlSessionFactory-mapper路径区分.*/
       public final static String MAPPER_PACKAGE = "com.grady.demotransaction.mapper";
       /**mapper.xml文件路径，必须与其他SqlSessionFactory-mapper路径区分.*/
       public final static String MAPPER_XML_PATH = "classpath:mapper/*.xml";
   
       @Value("${spring.datasource.url}")
       private String datasourceUrl;
   
       @Value("${spring.datasource.driver-class-name}")
       private String driverClassName;
   
       @Value("${spring.datasource.username}")
       private String userName;
   
       @Value("${spring.datasource.password}")
       private String password;
   
   
       @Bean(name = "dataSource")
       public DataSource dataSource() {
           //建议封装成单独的类
           DruidDataSource dataSource = new DruidDataSource();
           dataSource.setUrl(datasourceUrl);
           dataSource.setDriverClassName(driverClassName);
           dataSource.setUsername(userName);
           dataSource.setPassword(password);
           return dataSource;
       }
   
   
       //默认Bean首字母小写，简化配置
       //将SqlSessionFactory作为Bean注入到Spring容器中，成为配置一部分。
       @Bean
       public SqlSessionFactory sqlSessionFactory() throws Exception {
           SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
           sqlSessionFactoryBean.setDataSource(dataSource());
           sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_XML_PATH));
           return sqlSessionFactoryBean.getObject();
       }
   }
   ```

   4. 这样就可以将本来都写在application.properties中的配置拆成多个了（这里是两个）

      ==application.properties==

      ```properties
      spring.main.allow-bean-definition-overriding=true
      server.port=8080
      mybatis.config-location=classpath:config/mybatis-config.xml
      mybatis.mapper-locations=classpath:mapper/*.xml
      ```

      

      ==datasource.properties==

      ```properties
      spring.datasource.driver-class-name=com.mysql.jdbc.Driver
      spring.datasource.url=jdbc:mysql://localhost/sampledb
      spring.datasource.username=root
      spring.datasource.password=123456
      ```

      