[RxJava](https://github.com/ReactiveX/RxJava)

### 关于核心类 ###
#### Flowable ####
```java
public abstract class Flowable<T> implements Publisher<T>{
  public static <T> Flowable<T> create(FlowableOnSubscribe<T> source, BackpressureStrategy mode) {
    return RxJavaPlugins.onAssembly(new FlowableCreate<T>(source, mode));
  }
  
  public final void subscribe(FlowableSubscriber<? super T> s) {
    subscribeActual(z);
  }
  protected abstract void subscribeActual(Subscriber<? super T> s);
}
```

```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

```java
public interface FlowableOnSubscribe<T> {
    void subscribe(@NonNull FlowableEmitter<T> emitter) throws Exception;
}
```

- 由AbstractFlowableWithUpstream 组成Flowable链
- 通过Flowable.create创建时，FlowableCreate封装FlowableOnSubscribe为第一个Flowable：真正的生产者（FlowableEmitter触发onNext等事件）。BaseEmitter继承了Subscription；

### 关于设计 ###
观察者模式
迭代器模式
- 装饰者模式<br>
Flowable向上游通过subscribe方法传递Subscriber时，会封装原始Subscriber以提供额外的能力，如FlowableObserveOn，FlowableSubscribeOn等
