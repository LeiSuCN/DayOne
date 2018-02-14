

#### [数据的发送流程](http://blog.csdn.net/chenxiang0207/article/details/14054545)
1. 应用程序首先将要发送的数据写入该进程的内存地址空间中。通常在程序开发中这只需要一般的运行时变量赋值即可。
2. 应用程序通过系统函数库接口（比如send函数）向内核发出系统调用，系统内核随后将这些数据从用户态内存复制到由内核维护的一段称为内核缓冲区的内存地址空间。这块地址空间的大小通常是有限的，所有要发送的数据将以队列的形式进入这里，这些数据可能来自多个进程，每块数据都有一定的额外记号来标记他的去向。
3. 当数据写入内核缓冲区后，内核会通知网卡控制器前来取数据，同时CPU转而处理其他进程。网卡控制器接到通知后，便根据网卡驱动信息得知对应内核缓冲区的地址，将要发送的数据复制到网卡的缓冲区。
4. 网卡缓冲区的数据需要发送到线路中，同时释放缓冲区来获得更多要发送的数据。但是只有二进制的数字信号才可以在线路中传输，所以这时候需要对数据进行字节到位的转换，然后将数据的每个位按照顺序依次发出。
5. 发送时网卡会使用内部特定的物理装置来生成可以传播的各种信号。比如在使用铜线线路时，网卡会根据“0”与“1”的变化产生不同的电信号；而使用光纤线路时，网卡会产生不同的光信号。

### [DirectBuffer的优势](https://www.zhihu.com/question/60892134)
1. 底层通过write、read、pwrite，pread函数进行系统调用时，需要传入buffer的起始地址和buffer count作为参数。如果使用java heap的话，我们知道jvm中buffer往往以byte[] 的形式存在，这是一个特殊的对象，由于java heap GC的存在，这里对象在堆中的位置往往会发生移动，移动后我们传入系统函数的地址参数就不是真正的buffer地址了，这样的话无论读写都会发生出错。而C Heap仅仅受Full GC的影响，相对来说地址稳定
2. JVM规范中没有要求Java的byte[]必须是连续的内存空间，它往往受宿主语言的类型约束；而C Heap中我们分配的虚拟地址空间是可以连续的，而上述的系统调用要求我们使用连续的地址空间作为buffer。


###
io.netty.channel.DefaultChannelPipeline
io.netty.util.concurrent.EventExecutor.DefaultChannelHandlerContext
形成链的是ChannelHandlerContext而不是ChannelHandler

###

EventLoop --->  EventLoopGroup  ---> EventExecutorGroup  -->  ScheduledExecutorService

                EventExecutor

EventLoop的本质：Executor

核心实现：SingleThreadEventExecutor


### 底层的Socket或ServerSocket的事件如何绑定到EventLoop上的？
- AddressResolver 地址解析；gRpc中也有类似概念：NameResolver。将schema与uri分离从而可扩展；




## 核心组件

* Channel. -> Socket .屏蔽了底层实现，提供了统一的编程模型。
* EventLoop.
* ChannelFuture.
* ChannelHandler.业务逻辑与网络处理代码的分离。
* ChannelPipeline.
* ChannelHandlerContext

```
Channel --+  
...       |
Channel --+-> EventLoop(Single Thread) --+  
                                         |
Channel --+                              |--> EventLoopGroup
          |                              |
Channel --+-> EventLoop(Single Thread) --+ 
...       |
Channel --+
```

Channel 1-->1 ChannelPipeline 1-->n ChannelHandler 1-->1 ChannelHandlerContext 


                                                   I/O Request
                                              via {@link Channel} or
                                          {@link ChannelHandlerContext}
                                                        |
    +---------------------------------------------------+---------------+
    |                           ChannelPipeline         |               |
    |                                                  \|/              |
    |    +---------------------+            +-----------+----------+    |
    |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  |               |
    |               |                                  \|/              |
    |    +----------+----------+            +-----------+----------+    |
    |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  .               |
    |               .                                   .               |
    | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
    |        [ method call]                       [method call]         |
    |               .                                   .               |
    |               .                                  \|/              |
    |    +----------+----------+            +-----------+----------+    |
    |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  |               |
    |               |                                  \|/              |
    |    +----------+----------+            +-----------+----------+    |
    |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
    |    +----------+----------+            +-----------+----------+    |
    |              /|\                                  |               |
    +---------------+-----------------------------------+---------------+
                    |                                  \|/
    +---------------+-----------------------------------+---------------+
    |               |                                   |               |
    |       [ Socket.read() ]                    [ Socket.write() ]     |
    |                                                                   |
    |  Netty Internal I/O Threads (Transport Implementation)            |
    +-------------------------------------------------------------------+
	
	
## NIO

### Channel(管道)和Buffer(缓冲器)
- **唯一** **直接** 与Channel交互的Buffer： ByteBuffer
### Selector
- Java NIO's selectors allow a single thread to monitor multiple channels of input.

### Zero-copy
- Performance is enhanced by allowing the CPU to move on to other tasks while data copies proceed in parallel in another part of the machine. Also, zero-copy operations reduce the number of time-consuming mode switches between user space and kernel space. 
- 目前只有在使用NIO和Epoll传输时才可以使用。input streams can support zero-copy through the java.nio.channels.FileChannel's transferTo() method if the underlying operating system also supports zero copy.
- [wikipedia](https://en.wikipedia.org/wiki/Zero-copy)

### Select, Poll, Epoll
- [select / poll / epoll: practical difference for system architects](https://www.ulduzsoft.com/2014/01/select-poll-epoll-practical-difference-for-system-architects/)

### channel vs stream
- Channel: A channel represents an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing.
- Buffer: A buffer is a linear, finite sequence of elements of a specific primitive type.
- OutputStream: This abstract class is the superclass of all classes representing an output stream of bytes. An output stream accepts output bytes and sends them to some sink.
- InputStream: This abstract class is the superclass of all classes representing an input stream of bytes.
- stream 将连接和操作，数据类型全部耦合在了一起；而nio中将连接，操作，数据通过channel，（WritableByteChannel／ReadableByteChannel），Buffer解耦，更加符合SRP的理念。


## Pipeline，ChannelHandlerContext
- Pipeline中包含ChannelHandlerContext的双向链表，每个ChannelHandlerContext封装了**一个**Handler
- Outbound是package的发送者：即write、read等请求包的发送者，Inbound是package的接受者<br>
Outbound并不是write，同理Inbound也不是read
- public interface ChannelHandlerContext 
	extends AttributeMap, ChannelInboundInvoker, ChannelOutboundInvoker<br>
所以ChannelHandlerContext可以传播请求,即Invoker，如同Pipline：<br>
 public interface ChannelPipeline
        extends ChannelInboundInvoker, ChannelOutboundInvoker, Iterable<Entry<String, ChannelHandler>>
- Invoker 命令的发起者
- io.netty.channel.DefaultChannelPipeline<br>
```
/**
 * The default {@link ChannelPipeline} implementation.  It is usually created
 * by a {@link Channel} implementation when the {@link Channel} is created.
 */
public class DefaultChannelPipeline implements ChannelPipeline {
...
    protected DefaultChannelPipeline(Channel channel) {
        this.channel = ObjectUtil.checkNotNull(channel, "channel");
        succeededFuture = new SucceededChannelFuture(channel, null);
        voidPromise =  new VoidChannelPromise(channel, true);

        tail = new TailContext(this);
        head = new HeadContext(this);

        head.next = tail;
        tail.prev = head;
    }
...
}
```

- io.netty.channel.Channel.Unsafe: Unsafe operations that should never be called from user-code. These methods are only provided to implement the actual transport

