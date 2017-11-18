
```
ManagedChannelBuilder --> ManagedChannelProvider --> ManagedChannel --> ClientCall
```
在ManagedChannelProvider中通过java.util.ServiceLoader发现具体实现：
META-INF/services/io.grpc.ManagedChannelProvider 包含具体实现：io.grpc.netty.NettyChannelProvider
