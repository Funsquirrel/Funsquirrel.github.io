layout:     post
title:     RxJava
subtitle:   compose
date:       2018-10-27
author:     FS

catalog: true
tags: RxJava

# compose和ObservableTransformer

```Java
public final <R> Observable<R> compose(ObservableTransformer<? super T, ? extends R> composer) {
    return wrap(((ObservableTransformer<T, R>) ObjectHelper.requireNonNull(composer, "composer is null")).apply(this)); //1
}


public static <T> Observable<T> wrap(ObservableSource<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    if (source instanceof Observable) {
        return RxJavaPlugins.onAssembly((Observable<T>)source);
    }
    return RxJavaPlugins.onAssembly(new ObservableFromUnsafeSource<T>(source));
}

public interface ObservableTransformer<Upstream, Downstream> {
    @NonNull
    ObservableSource<Downstream> apply(@NonNull Observable<Upstream> upstream);
}

```
这是一些关键的源码，逻辑很清晰。应用场景请看这篇
[不要破坏链：使用RxJava的compose（）运算符](https://blog.danlew.net/2015/03/02/dont-break-the-chain/)。
ObservableTransformer中的apply方法接收一个Observable对象返回一个Observable对象，在这里就可以对Observable进行一些操作，当然返回的仍然时Observable。当结合compose操作符时，可以看到wrap()方法中apply(this)，这个this就是调用compose()的那个Observable,经过线程变换之后被返回。可以看出compose的作用就是把经过包装的Observable取出并且能延续链式调用。
