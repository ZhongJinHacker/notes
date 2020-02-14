##前言

###痛点

**时至今日，我们通常会使用应用程序或第三方库去提供通信功能。比如：我们通常使用HTTP客户端库去Web服务器检索信息;通过web服务调用一个远程程序。然而，一个通用协议或者它的实现往往不能适配的很好。就像我们不会使用通用的Http服务去交换一个大文件、电子邮件消息和近实时消息（比如金融信息和多人游戏数据 ）。一些致力于特殊目的的场景需要一个高度优化的协议实现，比如你可能想实现一个基于AJAX的用来聊天的HTTP服务器程序，流媒体HTTP服务器程序或大文件传输HTTP服务器。你有时甚至想为你的需求精确适配设计和实现一整个新的协议。另一个不可避免的场景是当你不对不处理一个遗留的专属协议去确保与一个旧系统互通。而在这个场景下最最重要的是我们可以多快递实现这个协议同时又不牺牲结果程序的健壮性和性能**



### 解决方案

**==Netty==是一个提供异步事件驱动的网络程序框架且可用于快速开发可维护的高性能、高扩展性的协议服务器和客户端的工具**

**另外，==Netty== 是一个NIO客户端或服务端框架，它可以非常快速而容易地开发网络程序，比如协议服务器和客户端。他可以非常简单而线性开发网络程序，比如TCP、UDP服务端程序。**

**’快速而简单‘并不意味着开发出来的程序会有难以维护或性能上的问题。Netty已非常仔细地设计且吸取了很多优秀协议的实现经验（比如 FTP、SMTP、HTTP和许多二进制和文本遗留协议），最终，Netty成功找到了一种没有丝毫妥协的方式来实现开发、性能、稳定性、灵活性的程序**

**一些用户可能已经找到了一些同样选择拥有这些优秀特性的网络应用框架，也许你会问Netty和他们有什么区别。答案是Netty的构建哲学。Netty从它被设计的第一天起就是为了给你从API和实现层面方面的最好的体验。它看不见摸不着，但你将会感受到它（的哲学）将会让你的工作十分地简单，就像你阅读的这篇用户指引和使用netty的过程一样**



### 开始使用

**这个章节将通过一些简单的例子介绍Netty的核心架构，来让你轻松入门。当你阅读完本章节，你将能基于Netty开发出一个客户端和服务端**



**如果你喜欢自上而下的学习方法，你可以从第二章节 架构概述 开始学习，然后回到这里**



### 开始前

**这里仅有两个用于运行本章节的最低要求；最新版本的Netty（4.x）和大于等于1.6版本的JDK。最新版本的Netty 在[项目下载页下载](https://netty.io/downloads.html)。请通过你喜欢的JDK供应商网站去下载正确的JDK版本**

**当你阅读时，你可能有很多关于本章介绍的类的问题。当你想获得更多信息，请参阅API指引，为了你的方便，所有用到的类都在线上API参阅文件中。另外，请不要犹豫在Netty的社区与我们联系，并让我们知道关于netty的问题，或者你的好的建议也可以告诉我们。**



### 开发一个取消服务程序

**世界上最简单的协议不是“Hello World”，而是取消。 取消协议就是不返回任何消息。**



**为了实现`取消`协议,唯一你要做的事情就是忽略所有你接受的消息。让我们直接从Netty操作IO事件的handler的实现开始**

```java
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. `DiscardServerHandler` extends `ChannelInboundHandlerAdapter`, `ChannelInboundHandlerAdapter`实现了`ChannelInboundHandler`.`ChannelInboundHandler`提供了一系列的事件处理方法，你可以重写它们。现在，你只要继承`ChannelInboundHandlerAdapter`就可以了，而不是自己实现handler 的interface
2. 我们重写`channelRead()`方法。这个方法将会用来接受消息（当有新数据从客户端发过来时）。在这个例子中，接收消息的类型是`ByteBuf`
3. 为了实现取消协议，这个handler就必须忽略掉所有接收消息。`ByteBuf`是一个引用计数对象，它通过直接调用`release()`方法来释放对象。请保持注意每个handler都有职责去释放每个流过此handler的引用计数对象。通常，`channelRead()`handler方法像下面这样被实现:

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```

4. 当在处理事件过程中由于IO错误或者handler的实现而触发一个异常时，`exceptionCaught()`事件方法将会被调用。大部分情况，这个异常将会被日志输出且相关联的channel将会被关闭，尽管方法的实现在处理该异常的情景时是不同的。举个例子来说，你可能想在关闭连接前发送一个带有错误码的响应消息。

至今为止，我们实现了取消服务的一半。现在剩下的一半将写在`mian()`方法，它将用来启动服务（服务中调用DiscardServerHandler）

```java
package io.netty.example.discard;
    
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port = 8080;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        }

        new DiscardServer(port).run();
    }
}
```



1. `NioEventLoopGroup`是一个多线程的事件循环，用来操作IO 事件。Netty提供一系列的`EventLoopGroup`实现用于不同的传输方式。我们现在要实现一个服务端的程序，所以需要两个`NioEventLoopGroup`。第一个通常被取名为`boss`，它用来处理接收新的连接。第二个通常被取名为`worker`，它用来处理接收那些已经被`boss`接收的且被boss注册到`work`上的连接。程序会创建多少线程和它们会如何与已创建的Channel之间映射取决于`EventLoopGroup`的实现和在构造时的配置
2. `ServerBootstrap`是一个帮助类，用于建议服务。你可以直接通过一个`Channel`建立一个服务。然而请注意那是一个很乏味的过程，在大多数情况下你不需要这么做。
3. 这里，我们明确使用`NioServerSocketChannel`(它专门被用来实例化一个新Channel)去接受一个新的连接
4. 一个特殊的handler将会经常被一个新的接受的`Channel`调用。`ChannelInitializer`就是这样的特殊的handler，它被用于帮助用户配置一个新的Channel。很可能有这种场景：你想为配置新Channel的`ChannelPipeline`，给它新增一些Handler，比如`DiscardServerHandler`，用于实现你的网络程序。当你的程序变得复杂，很可能你需要添加更多的handler到pipeline和抽象一个匿名类到顶级类中。
5. 你也可以为`Channel`的实现设置一些参数。我们写的是TCP/IP服务，所以我们被允许设置Socket的选项，比如`tcpNoDelay`和`keepAlive`。请阅读`ChannelOption`的API手册和指定的`ChannelConfig`实现去获得相关支持的概述
6. 你是否注意到`option()` 和`childOption()`？ `option()`用来设置`NioServerSocketChannel`。`childOption()`用来设置被父`ServerChannel`接受的`Channel`
7. 现在准备开始，剩下的部分就是绑定端口和启动服务类。现在，我们绑定`8080`端口。你可以随时调用`bind()`方法

祝贺你！你已经基于Netty完成了你的第一个服务。



### 研究收到的数据

现在我们已经写下了我们的第一个服务，我们需要测试一下它是否真的可以工作，最简单的方式是使用`telnet`命令来测试。比如，你可以在终端输入`telnet localhost 8080`然后输入一些东西。



然而，我们可以说服务真正的工作起来了吗？我不能知道，因为它是一个取消服务。你将不会收到任务响应。为了证明它真正的在工作，让我们来修改服务，让它打印它收到的消息。



我们已经知道无论任何时候收到数据时`channelRead()`方法将会被调用。让我们写下一些代码到`DiscardServerHandler`的`channelRead()` 方法中：

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) { // (1)
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg); // (2)
    }
}
```

（1）那个效率低下的循环可以被简化为：

```java
System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))
```



（2）或者, 你可以在这里使用`in.release()`

如果你在试运行`telnet`命令，你将看到服务端打印它收到的消息

这整个取消服务源码位于发布包的`io.netty.example.discard`包中



### 写一个Echo服务

目前为止，我们已经消费了数据，但是没有响应任何数据。然而一个服务通常被认为是会响应请求的。让我们学习一下如何实现`Echo`协议来一个返回收到的消息给客户端。

与三个章节的取消服务不同点是它会发送收到的数据，而不是在终端打印收到的数据。因此，只要再次修改`channelRead`方法就足够了。

```java
 @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```

(1) 一个`ChannelHandlerContext`对象提供了一系列的使你能出发一系列IO事件和操作的方法。现在，我们调用`write(Object)`方法来逐字写入我们收到的消息。请注意，在这里我们不会释放收到的消息（不像在取消服务的例子中我们释放了消息）。因为Netty会在消息写入发送的通道线后帮我们释放它

(2)`ctx.write(Object)`不会使消息写入到发送通道线。而是会在内部缓存起来，然后当调用`ctx.flush()`后刷入发送通道线。又或者，你可以调用更加简洁的方法`ctx.writeAndFlush(msg)`

如果你再次运行telnet命令，你将看到服务端将返回你发送过去的数据

这整个源码位于发布包的`io.netty.example.discard`包中



### 写一个时钟服务

本章节要实现的协议是`TIME`协议。与之前的例子不同的地方是它发送的消息是一个32bit的整型数据，不论收到任何数据且当发送完数据后立刻关上连接。举个例子：你将会学习到如何去构造和发送消息，和如何在完成后关闭连接。



因为我们将忽略任务接受到的数据，而是选择在建立连接是发送一个数据，所以我们此时不能使用`channelRead()`方法，而是选择重写`channelActive`方法。下面就是它的实现：

```java
package io.netty.example.time;

public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));
        
        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

(1)作为解释，当连接建立且准备好通信时，`channelActive()`方法将会被调用。让我们在这个方法里写个代表当前时间的32位整型数据

(2)为了发送一个新消息，我们需要创建一个缓冲区(buffer)，它用来包含消息。我们将写一个32位整型数据，因此我们需要一个容量至少位4字节的`ByteBuf`。可以通过`ChannelHandlerContexy.alloc()`来获得当前的`ByteBufAllocator`， 然后分配一个新的缓存空间

(3)像往常一样，我们写一个已构造的消息

但是稍等一下，`flip`方法在哪里呢？ 是不是我们在NIO模型中，在发送消息前不必使用`java.nio.ByteBuffer.flip()`呢？`ByteBuf`没有这个方法(`flip()`) ，原因它拥有两个指针：一个是因为读操作的，一个是因为写操作的。当你向 `ByteBuf`中写入数据时，写操作索引会增长，而此时读操作索引不会有任何变化。读操作索引和写操作索引分别代表着消息的开始位置和结束位置

相反地，如果(NIO编程中)不调用`flip()`方法，NIO 的缓冲区并没有提供一个很清晰的方式去指出消息内容的开始位置和结束位置。当你忘记调用`flip()`来发送缓冲区数据时，你将陷入麻烦中，因为没有数据或者错误的数据将会被发送。这样的措施是不会发生在基于Netty的编程中的，因为我们有为例不同操作而设置的不同指针。你将会发现当你使用它们时，它会使工作变得更加简单——一个不需要`flip()`的世界

另一个需要被注意的点是`ChannelHandlerContexy.write()`(和`writeAndFlush()`)方法将会返回一个`ChannelFuture`对象。一个`ChannelFuture`对象代表这个一个(可能)还未完成的IO操作。它意味着，任何任务请求执行的操作可能还未被执行，因为在Netty中所有的操作都是异步的。举个例子来说，下面的代码可能会在消息发送前关闭掉了连接：

```java
Channel ch = ...;
ch.writeAndFlush(message);
ch.close();
```

因此，你需要在`ChannelFuture`(在调用`write()`返回的)完成后再调用`close()` 方法。它在它的写操作完成时将会通知它所有的监听器。请一定注意，`close()`方法也不会立刻关掉连接，且也会返回一个`ChannelFuture`。

(4) 当一个写请求结束，我们要如何获得通知呢？只要在返回的`ChannelFuture`中添加一个 `ChannelFutureListener`就可以了。在这里，我们创建一个匿名的`ChannelFutureListener`当操作完成时关闭`Channel`

或者，你可以用预定义的Listener来简化的代码

```java
f.addListener(ChannelFutureListener.CLOSE)
```

你可以使用Unix命令`rdate`来测试我们时`TIME`服务是否按照期望在工作。

```shell
$ rdate -o <port> -p <host>
```

<port> 就是你在`main()`中指明的，<host>常常时localhost



### 写一个`TIME`客户端

不像==取消==和==ECHO==服务，我们需要一个`TIME`协议的客户端，因为人类是无法将一个32位的二进制数据转化为一个可读的日期数据的。本章节中，我们将讨论怎么确认服务在正确地工作，和学习如何用Netty写一个客户端。

用Netty开发服务端和客户端最大的不同是使用了不同实现的`Bootstrap`和`Channel`.请仔细看下面的代码：

```java
package io.netty.example.time;

public class TimeClient {
    public static void main(String[] args) throws Exception {
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            Bootstrap b = new Bootstrap(); // (1)
            b.group(workerGroup); // (2)
            b.channel(NioSocketChannel.class); // (3)
            b.option(ChannelOption.SO_KEEPALIVE, true); // (4)
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });
            
            // Start the client.
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

（1）`Bootstrap`和`ServerBootstrap`很类似，除了它是为了服务于 非服务端Channel，比如: 客户端或无连接的`Channel`

(2) 只有一个`EventLoopGroup`,它将用来作为boss group 和worker group的合体。boss 的gourp不会在客户端侧使用

(3) 与`NioServerSocketChanel`不同，`NioSocketChannel`专门用来创建一个客户端侧的`Channel`。

(4)注意不像使用`ServerBootstrap`时,这里我们没有使用`childOption()`，因为客户端的`SocketChannel`没有父容器

(5)我们需要调用`connect()`方法，而不是使用`bind()`方法



就像你看到的，它与服务端代码并没有很大的不同。那关于`ChannelHandler`的实现呢？它需要从服务端接受一个32位整型，并解释成一个人类可读的格式，打印解释好的时间，并再关闭连接：

```java
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg; // (1)
        try {
            long currentTimeMillis = (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```



(1)在TCP/IP 协议中，Netty从另一端发送的数据中读取数据，并写到`ByteBuf`中。



这个看起来非常简单，且看起来跟服务端的例子没什么区别。

然而，这个Handler有时会触发`IndexOutOfBundsException`而无法工作。我们将在下节讨论其中的原因



### 处理一个基于流的传输

#### 一个套接字缓冲的小警告

在一个基于流的传输中，比如TCP/IP，收到的数据都被缓存到一个套简直接收缓冲区中。不幸的是，这个缓冲区并不是一个包的队列，而是一个字节的队列。这意味着，即使你收到了两个个消息，且两个消息在彼此独立的包中，操作系统还是不会把它们当作是两个消息，而是当作是一堆字节。因此，不能保证你直接读到的数据就是你远端写入的数据。比如： 我们假设操作系统的TCP/IP栈接受到了三个包：

![](https://github.com/djxhero/some_little_thing/raw/master/res/images/netty/1.png)

由于这是一个基于流的协议的属性，这里在你的程序中会有很高的概率会读成以下片段的样子：

![](https://github.com/djxhero/some_little_thing/raw/master/res/images/netty/2.png)

因此，不管是是服务端还是客户端的接收数据的部分都需要整理接收到的数据到一个或多个有意义的帧中，这些帧可容易地被程序的业务逻辑理解。在上面例子中，这个接收的数据应该被帧华成如下这样：

![](https://github.com/djxhero/some_little_thing/raw/master/res/images/netty/3.png)



### 第一个版本

现在让我们再看`TIME`客户端的例子。我们同样有这个问题。一个32位的整型是一个非常小的数据，且不会经常被切分。然而，这个问题是它可以被切分，且切分的可能性会睡着数据交换的次数的升高而升高最简单的解决方案是创建一个内部累积的缓冲区，然后等待直到收到4个字节到内部缓冲区中。下面是一个修改过并修复了上述问题的`TimeClientHandler`的实现：

```java
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private ByteBuf buf;
    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

(1)一个`ChannelHandler`有两个生命周期回调方法：`handlerAdded()`和`handlerRemoved()`。你可以在其中执行初始化或反初始化的任务，只要任务不会阻塞太长的时间

(2) 第一，所有收到的数据需要写入`buf`中

(3) 然后，这个handler需要检查`buf`是否已含有足够的数据，本例中是4个字节，然后执行实际的业务逻辑。否则，Netty将会在收到更多数据后再次调用`channelRead()` 直到最后积累了4个字节



### 第二个解决方案

尽管第一个解决方案己经解决了`TIME`协议客户端的问题，但是代码看起来很丑陋。想象一下如果一个拥有一大堆字段的更复杂协议，像一个变长字段呢？那你的`ChannelInboundHandler`的实现将很快变得难以维护。

如果你已经注意到了这个问题，你可以多添加一个`ChannelHandler`到`ChannelPipeline`中，这样你可以一个整体的`ChannelHandler`拆分成多个模块部分来减少程序的复杂性。比如，你可以将`TimeClientHandler`分成两个handler：

- `TimeDecoder`用来处理直接流被碎片化的问题
- 然后`TimeClientHandler`将编程最初的最简单的版本

幸运的是，Netty体供了一个开箱即用的可扩展的类来帮助你写第一个版本

```java
package io.netty.example.time;

public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        
        out.add(in.readBytes(4)); // (4)
    }
}
```

(1) `ByteToMessageDecoder`实现了`ChannelInboundHandler`接口，它可以事我们处理流量碎片化的问题更加容易。

(2)当接收到新数据时，`ByteToMessageDecoder`会调用 `decode()`方法，同时会传入一个内部维护的累积缓冲区

(3)当接收到的数据不足时，`decode()`将不会添加任何东西到`out`中

(4)如果`decode()`方法中添加了一个对象到`out`中，那意味着解码器(`TimeDecoder`)成功解码了一条消息



现在我们拥有了另外一个handler了，需要将它添加到`ChannelPipeline`中，我现在需要修改`TimeClient`中`ChannelInitializer`的实现：

```java
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

如果你是一个敢于冒险的人，你可以尝试一下使用`ReplayingDecoder`，它将简化解码的过程。你可以咨询API文档来获得更多的信息。

```java
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```

另外， Netty提供了许多开箱即用的解码器，使你可以很容易地实现大部分协议，帮你避开整体化难维护的handler的实现方式。请参阅下面的包来获得更多详细的例子：

- `io.netty.example.factorial`是关于二进制协议
- `io.netty.example.telnet`是关于基于文本行协议



### 使用POJO来代替`ByteBuf`

当前我们看过的所有例子都是使用`ByteBuf`作为协议消息的主要数据结构。在本例中，我们将提高一下`TIME`协议的客户端和服务端例子，我们将使用一个POJO来代替`ByteBuf`。

在你的`ChannelHandler`中使用POJO的优势是十分明显的；当你把从`ByteBuf`中获取数据的部分抽离出来，你的handler将会变得具有高维护性和重用性。在`TIME`协议的客户端和服务端例子中，我们仅仅是读取一个32位整型数据，在直接使用`ByteBuf`上也没什么主要问题。然而，当你在实现一个真实协议时，你会发现做这种切分时十分有必要的。

首先，让我们定一个叫`UnixTime`的新类型

```java
package io.netty.example.time;

import java.util.Date;

public class UnixTime {

    private final long value;
    
    public UnixTime() {
        this(System.currentTimeMillis() / 1000L + 2208988800L);
    }
    
    public UnixTime(long value) {
        this.value = value;
    }
        
    public long value() {
        return value;
    }
        
    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}
```



现在我们重构一下`TimeDecoder`，用`UnixTime`替代`ByteBuf`。

```java
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```

 

当我们更新了解码器后，`TimeClientHandler`将不在需要用`ByteBuf`了：

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```

是不是更加地简单而优雅？当然可以将同样的技术应用个在服务侧。让我们先来更新一下`TimeServerHandler`：

```java
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```



现在我们仅仅是缺少一个编码器，它实现了`ChannelOutboundHandler`接口，用来将一个`UnixTime`对象解释成字节流后反写入`ByteBuf`中。这个操作要比写一个解码器要容器得多。因为在对消息编码时，你不必考虑字节碎片化和汇编到一起的问题。

```java
package io.netty.example.time;

public class TimeEncoder extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt((int)m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```

(1) 这一行是一个非常重要的。

​	首先，当我们编码数据到发送线时，我们传入一个初始的`ChannelPromise`以便Netty可以用它来标记是执行成功还是失败。

​	然后，我没不会去调用`ctx.flush()`。因为这个方法在这里会企图覆盖掉真正的 `flush()`操作。



更简单的操作是，你可以使用`MessageToByteEncoder`:

```java
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int)msg.value());
    }
}
```



最后要做的就是把`TimeEncoder`加入到服务端的`ChannelPipeline`中，且在`TimeServerHandler`之前。这是一项很小的工作了。

### 关闭你的程序

关闭Netty程序常常非常的简单，只要通过`shutdownGracefully()`关闭所有的`EventLoopGroup`就可以了。它会返回一个`Future`对象用来通知你`EventLooGroup` 和它所属的`Channel`全部关闭完毕。



### 总结

这个章节中，我们通过基于Netty写了一个完全可以工作的程序翻译来快速地浏览了Netty的知识。

还会有更多详细的知识在即将到来的章节中，我们也十分希望你可以去阅读在 `io.netty.example`包中的例子。

请主意，社区一直在等待你的问题和建议来提高你的Netty水平，和通过你的反馈来提升Netty及其文档。