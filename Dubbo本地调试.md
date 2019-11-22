```xml
    <dubbo:reference id="transferTimingUploadHisRPCService"
                     url="dubbo://100.118.67.3:20942"
                     interface="com.sf.idspTransferService.service.TransferTimingUploadHisRPCService"/>
```

只需要在需要引用的接口上加上

```xml
url="dubbo://100.118.67.3:20942"
```

#### 注意

### 这里必须是本机的IP

==localhost 和 127.0.0.1 是不生效的！！！！==

