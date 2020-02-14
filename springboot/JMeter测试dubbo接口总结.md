### Jmeter 测试dubbo 接口



#### 1. 安装JMeter

安装到/usr/local下

#### 2. github上下载 jmeter-plugins-dubbo-x.x.x-jar-with-dependencies.jar

​	将该jar 放到 jmeter安装目录的`lib/ext` 目录下

​	我的jmeter安装位置为`/usr/local/apache-jmeter`

​	这样jmeter就支持dubbo协议的请求了



#### 3. 打开jmeter

#### 4. 在  `Test Plan`下添加一个 线程组

#### 5. 右击 线程组 `Add -> Sampler -> Dubbo Sample`

#### 6. 如果注册中心用的是`Zookeeper` ，在`Registry Setting`中选择协议：`zookeeper` 和配置Zookeeper的地址和端口，==如果想直连本地的服务，协议选择 `none`, 然后address 填写 127.0.0.1 和dubbo的端口（这里可能自定义了端口，要填正确的端口号）==

#### 7. RPC Protocol Setting 中选择协议为dubbo协议

#### 8. 配置Interface Settting

 ```properties
 Interface： 填写dubbo服务的接口全路径名

		Method：填写要调用的方法

		  Args： 配置请求的参数

					   json格式参数： parmaType 填写方法入参的类名（VO）；paramValue 填写json格式数据

				     非json格式参数： 就是一个一个填，一个参数一行
 ```



####9. 添加 观察结果树 （==View Result Tree==）

​		每次点击==start图标==时，请求的结果集会在这里面展示