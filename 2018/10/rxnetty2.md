
# [RxNetty2](https://github.com/ReactiveX/RxNetty)

## 1. 简介

## 2. 代码拆解

### 2.1. 核心接口

- [Connection](https://github.com/ReactiveX/RxNetty/blob/0.5.x/rxnetty-common/src/main/java/io/reactivex/netty/channel/Connection.java): An abstraction over netty's channel providing Rx APIs. 核心接口
```
ChannelOperations
|
|—— Connection
```
- [ConnectionProvider](https://github.com/ReactiveX/RxNetty/blob/0.5.x/rxnetty-common/src/main/java/io/reactivex/netty/client/ConnectionProvider.java)创建Connection
```
Observable<Connection<R, W>> newConnectionRequest();
```
- [ConnectionProviderFactory](https://github.com/ReactiveX/RxNetty/blob/0.5.x/rxnetty-common/src/main/java/io/reactivex/netty/client/ConnectionProviderFactory.java)
```
ConnectionProvider<W, R> newProvider(Observable<HostConnector<W, R>> hosts);
```
