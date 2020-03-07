#### 在 nginx.conf 中配置以下内容

```nginx
...
http {
  
    ...
    server {
        # 这里表示upstream 的连接、读取、发送超时时间都是300秒
    	  proxy_connect_timeout 300;
        proxy_read_timeout    300;
        proxy_send_timeout    300;
        send_timeout          300;
    
        ...
      location / {
         ...
    }
  }
}
```

