
```
ManagedChannelBuilder --> ManagedChannelProvider --> ManagedChannel --> ClientCall --> ClientInterceptor
```

ClientCall为异步调用
```java
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

ManagedChannelImpl 构造函数
```java
  ManagedChannelImpl(
      AbstractManagedChannelImplBuilder<?> builder,
      ClientTransportFactory clientTransportFactory,
      BackoffPolicy.Provider backoffPolicyProvider,
      ObjectPool<? extends Executor> oobExecutorPool,
      Supplier<Stopwatch> stopwatchSupplier,
      List<ClientInterceptor> interceptors
  ){
    this.target = checkNotNull(builder.target, "target");
    this.nameResolverFactory = builder.getNameResolverFactory();
    this.nameResolverParams = checkNotNull(builder.getNameResolverParams(), "nameResolverParams");
    // target + ResolverFactory + ResolverParams ==> NameResolver
    // 获取到DnsNameResolver
    this.nameResolver = getNameResolver(target, nameResolverFactory, nameResolverParams); 
    // BalancerFactory为默认的PickFirstBalancerFactory
    // 在AbstractManagedChannelImplBuilder.DEFAULT_LOAD_BALANCER_FACTORY 定义
    this.loadBalancerFactory =
        checkNotNull(builder.loadBalancerFactory, "loadBalancerFactory");
    // GrpcUtil.SHARED_CHANNEL_EXECUTOR
    // SHARED_CHANNEL_EXECUTOR 为 Executors.newCachedThreadPool
    this.executorPool = checkNotNull(builder.executorPool, "executorPool");
    // GrpcUtil.SHARED_CHANNEL_EXECUTOR
    this.oobExecutorPool = checkNotNull(oobExecutorPool, "oobExecutorPool");
    this.executor = checkNotNull(executorPool.getObject(), "executor");
    /**
     * io.grpc.internal.ChannelExecutor
     * The thread-less Channel Executor used to run the state mutation logic in ManagedChannelImpl, InternalSubchannel and io.grpc.LoadBalancer s.
     *
     * <p>Tasks are queued until drain is called.  Tasks are guaranteed     to be run in the same
     * order as they are submitted.
     */
     // executor在reprocess中用来createRealStream
     // channelExecutor 是否实现了类似nodejs的EventLoop模型？
    this.delayedTransport = new DelayedClientTransport(this.executor, this.channelExecutor);

    /**
     * io.grpc.internal.DelayedClientTransport
     * A client transport that queues requests before a real transport is     available. When reprocess is called, this class applies the provided link SubchannelPicker} to pick a
     * transport for each pending stream.
     *
     * <p>This transport owns every stream that it has created until a real     transport has been picked
     * for that stream, at which point the ownership of the stream is transferred     to the real transport,
     * thus the delayed transport stops owning the stream.
     */
    this.delayedTransport.start(delayedTransportListener);

  /**
    * io.grpc.internal.BackoffPolicy
    * Determines how long to wait before doing some action (typically a retry, or a reconnect).
    */  
    this.backoffPolicyProvider = backoffPolicyProvider;
    this.transportFactory =
        new CallCredentialsApplyingTransportFactory(clientTransportFactory, this.executor);
    // RealChannel 调用transportFactory创建ClientCallImpl
    this.interceptorChannel = ClientInterceptors.intercept(new RealChannel(), interceptors);
    this.stopwatchSupplier = checkNotNull(stopwatchSupplier, "stopwatchSupplier");
    if (builder.idleTimeoutMillis == IDLE_TIMEOUT_MILLIS_DISABLE) {
      this.idleTimeoutMillis = builder.idleTimeoutMillis;
    } else {
      checkArgument(
          builder.idleTimeoutMillis
              >= AbstractManagedChannelImplBuilder.IDLE_MODE_MIN_TIMEOUT_MILLIS,
          "invalid idleTimeoutMillis %s", builder.idleTimeoutMillis);
      this.idleTimeoutMillis = builder.idleTimeoutMillis;
    }
    this.fullStreamDecompression = builder.fullStreamDecompression;
    this.decompressorRegistry = checkNotNull(builder.decompressorRegistry, "decompressorRegistry");
    this.compressorRegistry = checkNotNull(builder.compressorRegistry, "compressorRegistry");
    this.userAgent = builder.userAgent;

    phantom = new ManagedChannelReference(this);
    logger.log(Level.FINE, "[{0}] Created with target {1}", new Object[] {getLogId(), target});

  }
```

- ManagedChannelImpl 通过 transportProvider 提供ClientTransport（The client-side transport typically encapsulating a single connection to a remote server）；
- transportProvider中通过subchannelPicker获取PickResult --> Subchannel;
- LbHelperImpl中更新subchannelPicker;
- exitIdleMode中启动nameResolver并注册NameResolverListenerImpl的listener

 
在ManagedChannelProvider中通过java.util.ServiceLoader发现具体实现：
META-INF/services/io.grpc.ManagedChannelProvider 包含具体实现：io.grpc.netty.NettyChannelProvider


ClientCall = Channel.newCall(MethodDescriptor<RequestT, ResponseT> methodDescriptor, CallOptions callOptions)

```
public final class MethodDescriptor<ReqT, RespT> {

  private final MethodType type;
  private final String fullMethodName;
  private final Marshaller<ReqT> requestMarshaller;
  private final Marshaller<RespT> responseMarshaller;
  private final @Nullable Object schemaDescriptor;
  private final boolean idempotent;
  private final boolean safe;
  
  ... ...
 }
 ```
 
 
 ```
  public interface Marshaller<T> {
  
    public InputStream stream(T value);

    public T parse(InputStream stream);
  }
  ```
  
 ```
 public final class CallOptions {

  private Deadline deadline;
 
 private Executor executor;

  private String authority;

  private CallCredentials credentials;

  private String compressorName;

  private Object[][] customOptions = new Object[0][2];

  private List<ClientStreamTracer.Factory> streamTracerFactories = Collections.emptyList();

  private boolean waitForReady;

  private Integer maxInboundMessageSize;

  private Integer maxOutboundMessageSize;
  
  ...
}
 ```

```
/**
 * Transports for a single {@link SocketAddress}.
 *
 * <p>This is the next version of {@link TransportSet} in development.
 */
@ThreadSafe
final class InternalSubchannel implements WithLogId {
  private static final Logger log = Logger.getLogger(InternalSubchannel.class.getName());

  private final LogId logId = LogId.allocate(getClass().getName());
  private final EquivalentAddressGroup addressGroup;
  private final String authority;
  private final String userAgent;
  private final BackoffPolicy.Provider backoffPolicyProvider;
  private final Callback callback;
  private final ClientTransportFactory transportFactory;
  private final ScheduledExecutorService scheduledExecutor;
  ...
 }
```

ClientTransport封装了底层连接：
NettyClientTransport




NettyClientTransport -> handler.startWriteQueue(channel)? -> NettyClientHandler.write -> NettyClientHandler.createStream -> NettyClientStream.setHttp2Stream(http2Stream) -> AbstractStream2.onStreamAllocated -> AbstractStream2.notifyIfReady -> ClientCallImpl.onReady -> SerializingExecutor.execute -> SerializingExecutor.schedule -> executor.execute
