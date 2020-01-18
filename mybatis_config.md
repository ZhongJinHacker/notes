```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <!-- 打印查询语句 -->
<!--        <setting name="logImpl" value="LOG4J"/>-->
        <setting name="logImpl" value="STDOUT_LOGGING" />
        <!-- 映射ID到对象 -->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>
    <!-- 配置分页插件 -->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <property name="offsetAsPageNum" value="true"/>
            <property name="rowBoundsWithCount" value="true"/>
        </plugin>
    </plugins>
</configuration>
```