### token 总结

#### 1. token 和SessionID 的区别

Token机制相对于Cookie机制又有什么好处呢？

- **支持跨域访问**: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.
- **无状态(也称：服务端可扩展行)**:Token机制在服务端不需要存储session信息，因为Token 自身**包含**了所有==登录用户的信息==(服务端设置的，前端不能设，也不能取)，只需要在客户端的cookie或本地介质存储状态信息.
- **更适用CDN**: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可.
- **解耦**: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.
- **更适用于移动应用**: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
- **CSRF**:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。
- **性能**: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多（<u>但token要比sessionID长，需占用更大贷款，也能容忍</u>）.
- **不需要为登录页面做特殊处理**: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.
- **基于标准化**:你的API可以采用标准化的 ==JSON Web Token (JWT)==. 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）.



### 基于JWT的Token认证机制实现

JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

### JWT的组成

一个**JWT**实际上**就是**一个**字符串**，它由**三部分**组成：==头部==、==载荷(也叫payload)==与==签名==。



**1.头部（Header）**

**base64编码**

头部用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等， 

```json
{
  "typ": "JWT",
	"alg": "HS256"
}
//在头部指明了签名算法是HS256算法。
//当然头部也要进行BASE64编码，编码后的字符串如下：
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```





**2. 载荷（payload) 样例：**

**base64编码**

这里里放声明内容，可以说就是存放沟通讯息的地方，在定义上有3种声明（Claims）

- **Registered claims（注册声明）**:
  这些是一组预先定义的声明，它们不是强制性的，但推荐使用，以提供一组有用的，可互操作的声明。 其中一些是：`iss`（发行者），`exp`（到期时间），`sub`（主题），`aud`（受众）等。[#Registered Claim Names#](https://tools.ietf.org/html/rfc7519#section-4.1)
- **Public claims（公开声明）**:
  这些可以由==使用JWT的人员随意定义==。 但为避免冲突，应在IANA JSON Web令牌注册表中定义它们，或将其定义为包含防冲突命名空间的URI。
- **Private claims（私有声明）**:
  这些是为了同意使用它们但是既没有登记，也没有公开声明的各方之间共享信息，而创建的定制声明。

```json
{
  "iss": "Online JWT Builder", 
  "iat": 1416797419, 
  "exp": 1448333419, 
  "aud": "www.example.com", 
  "sub": "jrocket@example.com", 
  "GivenName": "Johnny", 
  "Surname": "Rocket", 
  "Email": "jrocket@example.com", 
  "Role": [ "Manager", "Project Administrator" ] 
}

//iss: 该JWT的签发者，是否使用是可选的；
//sub: 该JWT所面向的用户，是否使用是可选的；
//aud: 接收该JWT的一方，是否使用是可选的；
//exp(expires): 什么时候过期，这里是一个Unix时间戳，是否使用是可选的；
//iat(issued at): 在什么时候签发的(UNIX时间)，是否使用是可选的；
//其他还有：
//nbf (Not Before)：如果当前时间在nbf里的时间之前，则Token不被接受；一般都会留一些余地，比如几分钟；，是否使用是可选的；
-------------------------------------------------------------------------------
//上面的JSON对象进行[base64编码]可以得到下面的字符串。这个字符串我们将它称作JWT的Payload
eyJpc3MiOiJKb2huIFd1IEpXVCIsImlhdCI6MTQ0MTU5MzUwMiwiZXhwIjoxNDQxNTk0NzIyLCJhdWQiOiJ3d3cuZXhhbXBsZS5jb20iLCJzdWIiOiJqcm9ja2V0QGV4YW1wbGUuY29tIiwiZnJvbV91c2VyIjoiQiIsInRhcmdldF91c2VyIjoiQSJ9
```



**3.签名（Signature）【重要】**

**HS256算法 + 秘钥 加密**

将上面的两个编码后的字符串都用句号.**连接**在一起（头部在前），就形成了:

```json
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcm9tX3VzZXIiOiJCIiwidGFyZ2V0X3VzZXIiOiJB.......
```



==最后==，我们将==上面拼接完的字符串==用==HS256算法==进行==**加密**==。在加密的时候，我们还需要提供一个密钥（secret）。如果我们用mystar作为密钥的话，那么就可以得到我们加密后的内容:

```json
rSWamyAYwuHCo7IFAgd1oRpSP7nzL7BF5t7ItqpKViM
```

这可签名非常重要，是防止伪造token的关键，因为前面都是

最后将这一部分签名也拼接在被签名的字符串后面，我们就得到了完整的JWT:

```json
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmcm9tX3VzZXIiOiJCIiwidGFyZ2V0X3VzZXIiOiJBIn0.rSWamyAYwuHCo7IFAgd1oRpSP7nzL7BF5t7ItqpKViM
```



### 4. 常见问题

#### ① JWT 安全嗎?

Base64编码方式是可逆的，也就是透过编码后发放的Token内容是可以被解析的。一般而言，我们都不建议在==有效载荷内放敏感讯息，比如使用者的密码==。

#### ② JWT Payload 內容可以被伪造嗎？

JWT其中的一个组成内容为Signature，可以防止通过Base64可逆方法回推有效载荷内容并将其修改。因为Signature是经由Header跟Payload一起Base64组成的(==Signature可以防止Header和payload被篡改==)。

#### ③ 如果我的 Cookie 被窃取了，那不就表示第三方可以做 CSRF 攻击?

是的，Cookie丢失，就表示身份就可以被伪造。故官方建议的使用方式是存放在LocalStorage中，并放在请求头中发送。

#### ④ 空间及长度问题？

JWT Token通常长度不会太小，特别是Stateless JWT Token，把所有的数据都编在Token里，很快的就会超过Cookie的大小（4K）或者是URL长度限制， ==所以只放少量必要信息==

#### ⑤ Token失效问题？

- 无状态JWT令牌（Stateless JWT Token）发放出去之后，不能通过服务器端让令牌失效，必须等到过期时间过才会失去效用。

- 假设在这之间Token被拦截，或者有权限管理身份的差异造成授权Scope修改，都不能阻止发出去的Token失效并要求使用者重新请求新的Token。

  

## 5. JWT使用建议

- 不要存放敏感信息在Token里（**因为前面两段是Base64编码的，是可逆的**）。
- Payload中的`exp`时效不要设定太长（token在请求头中，被他人获取可直接使用）。
- 开启`Only Http`预防XSS攻击。
- 如果担心重播攻击（replay attacks ）可以增加`jti`（JWT ID），`exp`（有效时间） Claim。
- 在你的应用程序应用层中增加黑名单机制，必要的时候可以进行Block做阻挡（这是针对掉令牌被第三方使用窃取的手动防御）。