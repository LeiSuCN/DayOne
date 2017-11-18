
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

ManagedChannelImpl 构造函数
```
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
    this.nameResolver = getNameResolver(target, nameResolverFactory, nameResolverParams); // 获取到DnsNameResolver
    // BalancerFactory为默认的PickFirstBalancerFactory，在AbstractManagedChannelImplBuilder.DEFAULT_LOAD_BALANCER_FACTORY 定义
    this.loadBalancerFactory =
        checkNotNull(builder.loadBalancerFactory, "loadBalancerFactory");
    // GrpcUtil.SHARED_CHANNEL_EXECUTOR：通过Executors.newCachedThreadPool创建池
    this.executorPool = checkNotNull(builder.executorPool, "executorPool");
    this.oobExecutorPool = checkNotNull(oobExecutorPool, "oobExecutorPool");
    this.executor = checkNotNull(executorPool.getObject(), "executor");
    this.delayedTransport = new DelayedClientTransport(this.executor, this.channelExecutor);
    this.delayedTransport.start(delayedTransportListener);
    this.backoffPolicyProvider = backoffPolicyProvider;
    this.transportFactory =
        new CallCredentialsApplyingTransportFactory(clientTransportFactory, this.executor);
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

ManagedChannelImpl 通过 transportProvider 提供ClientTransport（The client-side transport typically encapsulating a single connection to a remote server）；
transportProvider中通过subchannelPicker获取PickResult --> Subchannel;
LbHelperImpl中更新subchannelPicker;
 
在ManagedChannelProvider中通过java.util.ServiceLoader发现具体实现：
META-INF/services/io.grpc.ManagedChannelProvider 包含具体实现：io.grpc.netty.NettyChannelProvider
