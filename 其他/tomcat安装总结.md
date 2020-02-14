#### 1. 进入tomcat官网下载 ==Core 下的 tar.gz包==

#### 2. 解压

```shell
tar -zxvf apache-tomcat-7.0.82.tar.gz
```

#### 3. 修改目录名称

```shell
mv apache-tomcat-7.0.82 apache-tomcat
```

#### 4. 放到usr/local下

```shell
sudo mv apache-tomcat /usr/local/
```

#### 5. 进入apache-tomcat， 执行bin下的startup.sh ，访问localhost:8080，看到tomcat页面即安装成功

```shell
cd /usr/local/apache-tomcat/bin
./startup.sh
```

