---
layout:     post
title:      RxJava
subtitle:   Observable
date:       2018-10-27
author:     FS

catalog: true
tags:
    - RxJava

---

# Observable源码分析




```Java
public interface ObservableSource<T> {

    /**
     * Subscribes the given Observer to this ObservableSource instance.
     * @param observer the Observer, not null
     * @throws NullPointerException if {@code observer} is null
     */
    void subscribe(@NonNull Observer<? super T> observer);
}
```


```Java
public abstract class Observable<T> implements ObservableSource<T> {
// 代码
}
```

```Java
// 这是Observable抽象类中唯一的抽象方法
protected abstract void subscribeActual(Observer<? super T> observer);
```
翻译为实际注册，那为什么要把这个方法抽象呢，接着往下看。

```Java
public final void subscribe(Observer<? super T> observer) {
    ObjectHelper.requireNonNull(observer, "observer is null");
    try {
        observer = RxJavaPlugins.onSubscribe(this, observer);

        ObjectHelper.requireNonNull(observer, "Plugin returned null Observer");

        subscribeActual(observer);
    } catch (NullPointerException e) { // NOPMD
        throw e;
    } catch (Throwable e) {
        Exceptions.throwIfFatal(e);
        // can't call onError because no way to know if a Disposable has been set or not
        // can't call onSubscribe because the call might have set a Subscription already
        RxJavaPlugins.onError(e);

        NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
        npe.initCause(e);
        throw npe;
    }
}
```
这是Observable类中的final方法，表示将Observer注册到Observable。可以看到会调用到subscribeActual(observer)方法。我们知道RxJava中有很多操作符比如create、defre、just，他们对应的类如ObservableCreate、ObservableDefer、ObservableJust都继承自Observable抽象方法，并覆写可以看到会调用到subscribeActual方法，添加一些操作。



