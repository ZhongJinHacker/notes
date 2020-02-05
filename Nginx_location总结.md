#### 最近操作Nginx.conf 的location部分，发现了一个巨坑，在这做个记录



#### 当我用一下配置时

```nginx
      location = / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
      }
```

#### 浏览器域名访问， ==死活读不到html内容==

##### 改成如下就好了

```nginx
      location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
      }
```

*为什么精确匹配不行？很迷*

#### 之后查看文档

```shell
匹配模式及顺序

匹配字符串分为两种：普通字符串（literal string）和正则表达式（regular expression），其中 ~ 和 ~* 用于正则表达式， 其他前缀和无任何前缀都用于普通字符串。

匹配顺序是：
1、先匹配普通字符串，将最精确的匹配暂时存储；
2、然后按照配置文件中的声明顺序进行正则表达式匹配，只要匹配到一条正则表达式，则停止匹配，取正则表达式为匹配结果；
3、如果所有正则表达式都匹配不上，则取1中存储的结果；
4、如果普通字符串和正则表达式都匹配不上，则报404 NOT FOUND。

location = /uri        =开头表示精确前缀匹配，只有完全匹配才能生效。

location ^~ /uri       ^~开头表示普通字符串匹配上以后不再进行正则匹配。

location ~ pattern     ~开头表示区分大小写的正则匹配。

location ~* pattern    ~*开头表示不区分大小写的正则匹配。

location /uri          不带任何修饰符，表示前缀匹配。

location /             通用匹配，任何未匹配到其他location的请求都会匹配到。

注意：正则匹配会根据匹配顺序，找到第一个匹配的正则表达式后将停止搜索。普通字符串匹配则无视顺序，只会选择最精确的匹配。
```

#### 也未找到答案，有可能是 ==/== 不能用==精确匹配==吧

