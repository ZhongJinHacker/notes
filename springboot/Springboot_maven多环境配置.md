### 开发过程中总是需要多环境配置，而Spring自带的方式不是那么优秀，可以利用maven来帮助做到



#### 可以再==pom.xml==中配置==profiles==来做到



#### 打包命令：

```shell
mvn clean package -Psit
```



**这里的==-P==对应的就是profiles对应的属性**



#### 然后再 build 标签的resource 标签中自定义资源路径，就可以实现多环境打包

##### resouce目录为：

```shell
resources
	- dev
	   - application.properties
	- sit
	   - application.properties
	...
```



##### 这样就可以将dev或者sit的配置文件按属性分开打包

```shell
这里我曾犯过一个错误：
我将sit下的application.properties 命名为 application-sit.properties

我以为执行 java -jar app.jar时会加载到application-sit.properties

而实际并不会；必须执行java -jar app.jar -Psit 才可以

所以是多此一举
```





#### pom.xml 例子如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.grady</groupId>
	<artifactId>fim</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>fim</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		...
	</dependencies>

	<profiles>
		<!-- 开发环境 -->
		<profile>
			<id>dev</id>
			<properties>
				<profiles.active>dev</profiles.active>
			</properties>
			<activation>
				<!-- 表示默认激活 -->
				<activeByDefault>true</activeByDefault>
			</activation>
		</profile>
		<!-- sit 环境 -->
		<profile>
			<id>sit</id>
			<properties>
				<profiles.active>sit</profiles.active>
			</properties>
		</profile>
	</profiles>

	<build>
		<finalName>fim-backend-${version}</finalName>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<executable>true</executable>
				</configuration>
			</plugin>
		</plugins>

		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<excludes>
					<exclude>dev/*</exclude>
					<exclude>sit/*</exclude>
				</excludes>
			</resource>
			<resource>
				<!-- 根据不同环境打包不同目录下的配置文件 -->
				<directory>src/main/resources/${profiles.active}</directory>
			</resource>
		</resources>
	</build>

</project>

```



