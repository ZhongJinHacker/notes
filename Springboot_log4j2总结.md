### Log4j2 

#### PS:  Log4j 已不再维护，而最新的是Log4j2， Log4j2 是全部重写了Log4j，并拥有更加优秀的性能



#### 1. 引入依赖，和去掉logging的依赖

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>

		<!-- log4j2 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j2</artifactId>
		</dependency>
```



#### 2. 在resources 下增加文件log4j2.xml，默认会读取这个命名的文件（名字也可以改，但没啥意义，默认就行）







==Log4j2.xml 解释版==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
    status : 这个用于设置log4j2自身内部的信息输出,可以不设置,当设置成trace时,会看到log4j2内部各种详细输出
    monitorInterval : Log4j能够自动检测修改配置文件和重新配置本身, 设置间隔秒数。
-->
<Configuration status="WARN" monitorInterval="60">
  <Properties>
    <!-- 配置日志文件输出目录 -->
    <Property name="LOG_HOME">./logs</Property>

    <!-- 格式化输出：%date表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
    <!-- %logger{36} 表示 Logger 名字最长36个字符 -->
<!--    <property name="LOG_PATTERN" value="%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />-->

    <!-- 这个多打印出了日期，执行的函数名及行号 -->
    <property name="LOG_PATTERN" value="%d|%t|%X{uid}|%-5level|%c.%M:%L|%m%n" />
  </Properties>

  <Appenders>

    <!-- 控制台输出配置 -->
    <Console name="Console" target="SYSTEM_OUT">
      <!-- 控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch) -->
      <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/>
      <!--输出日志的格式-->
      <PatternLayout pattern="${LOG_PATTERN}"/>
    </Console>

    <!-- FileAppender。servlet容器中的两个web应用程序
        可以拥有它们各自的配置，如果Log4j2位于它们共同使用
        的类加载器中，则可以安全地将日志写入同一个文件。默认
         启用bufferedIO以及immediateFlush（前者提高性能，后
         者可以保证写入，缓冲区没满也写入！每次写操作都会调
         用flush，不过有点影响性能），如果是异步Logger，即便
         immediateFlush设置为false，异步Logger和appender也
         将在一批事件结束时自动flush，这样做比较高效同时
         保证了数据写入磁盘 -->
    <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，适合临时测试用-->
    <File name="Debug" fileName="${LOG_HOME}/debug.log" append="false">
      <PatternLayout pattern="${LOG_PATTERN}"/>
    </File>

    <!-- 似乎可以看作是FileAppender的进化版，实现上不大
        一样，根据报导，相比启用了bufferedIO的FileAppender
        性能提高了20-200%（RandomAccessFile没有bufferedIO
        这个选项，因为它总是buffered的！） -->
    <RollingRandomAccessFile name="Error" immediateFlush="true" bufferSize="4096" fileName="${LOG_HOME}/error.log" filePattern="${LOG_HOME}/error.log.%d{yyyy-MM-dd}.gz" ignoreExceptions="false">
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Filters>
        <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
      </Filters>
      <!-- 这个的默认是一小时触发一次 -->
      <TimeBasedTriggeringPolicy/>
      <DefaultRolloverStrategy>
        <Delete basePath="${LOG_HOME}" maxDepth="2">
          <IfFileName glob="error.log.*.gz"/>
          <IfLastModified age="30d"/>
        </Delete>
      </DefaultRolloverStrategy>
    </RollingRandomAccessFile>

    <RollingRandomAccessFile name="System" immediateFlush="true" bufferSize="4096"
                             fileName="${LOG_HOME}/system.log"
                             filePattern="${LOG_HOME}/system.log.%d{yyyy-MM-dd}.gz"
                             ignoreExceptions="false">
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Filters>
        <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/>
      </Filters>
      <!-- 表示两天触发一次策略 -->
      <Policies>
        <CronTriggeringPolicy schedule="0 0 2 * * ?" evaluateOnStartup="true"/>
      </Policies>

      <DefaultRolloverStrategy>
        <Delete basePath="${LOG_HOME}" maxDepth="2" followLinks="true">
          <IfFileName glob="system.log.*.gz"/>
          <IfLastModified age="7d"/>
        </Delete>
      </DefaultRolloverStrategy>
    </RollingRandomAccessFile>

  </Appenders>

  <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
  <!-- 定义logger，只有定义了logger 并引入appender，appender才会生效 -->
  <Loggers>
    <!--若是additivity设为false，则子Logger 只会在自己的appender里输出，而不会在 父Logger 的appender里输出。-->
    <!--过滤掉mybatis的一些无用的DEBUG信息-->
    <Logger name="org.mybatis" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>

    <!--过滤掉spring的一些无用的DEBUG信息-->
    <Logger name="org.springframework" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>

    <Logger name="org.hibernate.validator" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>

    <!-- Root节点用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出 -->
    <!-- 建立一个默认的root的logger -->
    <Root level="trace" includeLocation="true">
      <AppenderRef ref="Console"/>
      <AppenderRef ref="Error" />
      <AppenderRef ref="System" />
      <AppenderRef ref="Debug" />
    </Root>
  </Loggers>
</Configuration>
```

