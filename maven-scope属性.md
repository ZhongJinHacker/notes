

### Maven 中的 scope 属性解释



```xml
        <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>el-impl</artifactId>
        --> <scope>provided</scope>
        </dependency>
```



scope 定义了类包在项目的使用阶段

项目阶段包括： 编译 运行 测试 和 发布



### 分类说明



#### complie

这个是默认的scope，表示该依赖会参与到编译、测试和运行阶段，==打包时也会打包进去==



#### test

==该依赖仅参与到测试阶段的编译和执行==



#### runtime

表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过编译

比如mysql 驱动依赖

#### provided

意味着打包的时候可以不用包进去，别的设施（web container）会提供。事实上该依赖理论上可以参与编译、测试、运行等周期。相当于compile，但是打包阶段做了exclude的动作



#### system



和provide相同，不过被依赖项不会从maven仓库抓，而是从本地系统文件拿，一定要配合systemPath使用





