```shell
 http://172.16.5.130:8080/swagger-ui.html
```

==上面的ip:port 根据实际情况调换==

```shell
如果设置了server.servlet.context-path

比如：
server.servlet.context-path=/chat
(注意如果配置这个参数，这里的“/”是必须的，否则报错)

那么swagger访问地址变为
 http://ip:port/chat/swagger-ui.html

```





