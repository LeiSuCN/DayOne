# zuul2

## 核心类
### [ProxyEndpoint](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/filters/endpoint/ProxyEndpoint.java)
核心转发类

### [BasicNettyOrigin](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/origins/BasicNettyOrigin.java)
对被代理服务器的抽象，通过[ClientChannelManager](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/netty/connectionpool/DefaultClientChannelManager.java)管链接
<br>


zuul2中默认的http客户端不再是ribbon而是改为了netty实现

### [DynamicServerListLoadBalancer](https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/DynamicServerListLoadBalancer.java)
动态服务发现类，通过初始化方法初始ServerList字段以及启动更新服务
```java
    public void initWithNiwsConfig(IClientConfig clientConfig) {
  ...
            super.initWithNiwsConfig(clientConfig);
            String niwsServerListClassName = clientConfig.getPropertyAsString(
                    CommonClientConfigKey.NIWSServerListClassName,
                    DefaultClientConfigImpl.DEFAULT_SEVER_LIST_CLASS);

            ServerList<T> niwsServerListImpl = (ServerList<T>) ClientFactory
                    .instantiateInstanceWithClientConfig(niwsServerListClassName, clientConfig);
            this.serverListImpl = niwsServerListImpl;

            if (niwsServerListImpl instanceof AbstractServerList) {
                AbstractServerListFilter<T> niwsFilter = ((AbstractServerList) niwsServerListImpl)
                        .getFilterImpl(clientConfig);
                niwsFilter.setLoadBalancerStats(getLoadBalancerStats());
                this.filter = niwsFilter;
            }

            String serverListUpdaterClassName = clientConfig.getPropertyAsString(
                    CommonClientConfigKey.ServerListUpdaterClassName,
                    DefaultClientConfigImpl.DEFAULT_SERVER_LIST_UPDATER_CLASS
            );

            this.serverListUpdater = (ServerListUpdater) ClientFactory
                    .instantiateInstanceWithClientConfig(serverListUpdaterClassName, clientConfig);

            restOfInit(clientConfig);
...
    }
```
### [ZuulServerChannelInitializer](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/netty/server/ZuulServerChannelInitializer.java)
Channel初始化

### [ZuulFilterChainHandler](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/netty/filter/ZuulFilterChainHandler.java)
调度类

### [ZuulFilterChainRunner](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/netty/filter/ZuulFilterChainRunner.java)
调度类

### [ZuulEndPointRunner](https://github.com/Netflix/zuul/blob/2.1/zuul-core/src/main/java/com/netflix/zuul/netty/filter/ZuulEndPointRunner.java)
调度类

##  测试代码
```java

        ConfigurationManager.loadCascadedPropertiesFromResources("application");

        EurekaInstanceConfig eurekaInstanceConfig = new MyDataCenterInstanceConfigProvider().get();

        /** para10: ApplicationInfoManager */
        ApplicationInfoManager applicationInfoManager = new ApplicationInfoManager(eurekaInstanceConfig, (ApplicationInfoManager.OptionalArgs)null);

        /** para1: ServerStatusManager*/
         ServerStatusManager serverStatusManager = new ServerStatusManager(applicationInfoManager, null);

        /** para2: FilterLoader*/
        FilterRegistry filterRegistry = new FilterRegistry();
        filterRegistry.put(RoutesFilter.class.getCanonicalName(), new RoutesFilter());
        FilterLoader filterLoader = new FilterLoader(filterRegistry, null, new DefaultFilterFactory());

        /** para6: Registry */
        Registry registry = new DefaultRegistry();

        /** para3: SessionContextDecorator */
        OriginManager originManager = new BasicNettyOriginManager(registry);
        SessionContextDecorator sessionContextDecorator = new ZuulSessionContextDecorator(originManager);

        /** para4: FilterUsageNotifier */
        FilterUsageNotifier filterUsageNotifier = new BasicFilterUsageNotifier();

        /** para5: RequestCompleteHandler */
        RequestCompleteHandler requestCompleteHandler = null;


        /** para7: DirectMemoryMonitor */
        DirectMemoryMonitor directMemoryMonitor = new DirectMemoryMonitor();

        /** para8: EventLoopGroupMetrics */
        EventLoopGroupMetrics eventLoopGroupMetrics = new EventLoopGroupMetrics(registry);

        /** para9: EurekaClient */
        EurekaClient eurekaClient = null;

        /** para11: AccessLogPublisher */
        AccessLogPublisher accessLogPublisher = null;

        SampleServerStartup startup =
                new SampleServerStartup(
                        serverStatusManager,
                        filterLoader,
                        sessionContextDecorator,
                        filterUsageNotifier,
                        requestCompleteHandler,
                        registry,
                        directMemoryMonitor,
                        eventLoopGroupMetrics,
                        eurekaClient,
                        applicationInfoManager,
                        accessLogPublisher
                );

        startup.init();

        Server server = startup.server();
        server.start(true);
```
```application.properties
api.ribbon.NFLoadBalancerClassName=com.netflix.loadbalancer.DynamicServerListLoadBalancer
api.ribbon.listOfServers=0.0.0.0:9482
```
