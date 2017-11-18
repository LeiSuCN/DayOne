
```
ManagedChannelBuilder --> ManagedChannelProvider --> ManagedChannel --> ClientCall --> ClientInterceptor
```

ClientCall为异步调用
```
public abstract class ClientCall<ReqT, RespT> {
    public ClientCall() {
    }

    public abstract void start(ClientCall.Listener<RespT> var1, Metadata var2);

    public abstract void request(int var1);

    public abstract void cancel(@Nullable String var1, @Nullable Throwable var2);

    public abstract void halfClose();

    public abstract void sendMessage(ReqT var1);

    public boolean isReady() {
        return true;
    }

    public void setMessageCompression(boolean enabled) {
    }

    public Attributes getAttributes() {
        return Attributes.EMPTY;
    }

    public abstract static class Listener<T> {
        public Listener() {}

        public void onHeaders(Metadata headers) {}

        public void onMessage(T message) { }

        public void onClose(Status status, Metadata trailers) {}

        public void onReady() { }
    }
}
```
ManagedChannel 的默认实现：io.grpc.internal.ManagedChannelImpl

ManagedChannelImpl 通过 transportProvider 提供ClientTransport（The client-side transport typically encapsulating a single connection to a remote server）；
transportProvider中通过subchannelPicker获取PickResult --> Subchannel;
LbHelperImpl中更新subchannelPicker;
 
在ManagedChannelProvider中通过java.util.ServiceLoader发现具体实现：
META-INF/services/io.grpc.ManagedChannelProvider 包含具体实现：io.grpc.netty.NettyChannelProvider
