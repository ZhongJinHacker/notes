### 使用Devtools



#### 1. 在module的 pom中加入

```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
      <optional>true</optional>
    </dependency>
```

#### 2. 在父pom中使用构建插件插件

```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

#### 3. 开启自动编译选项

```xml
idea -> settings -> compile 
勾选
1. Automatically show first error in editor
2. Display notification on build completion
3. Build project automatically
4. Compile independent modules in parallel
```

#### 4. 使用快捷键（Mac）开启热部署

```xml
Command + Shift + Option + /
```

##### 选择`Registry`

##### 勾选`compile.automake.allow.when.app.running`

##### 勾选`actionSystem.assertFocusAccessFromEdt`

#### 5. 重启IDEA



### 建议还是不开启，觉得耗资源

