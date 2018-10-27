layout:     post
title:      RxJava
subtitle:   defer
date:       2018-10-27
author:     FS

catalog: true
tags:
    - RxJava

# defer

```Java
public class Main {

    public static void main(String[] args) {

        Main main = new Main();
        main.doSomeWork();
    }

    private void doSomeWork() {
        Car car = new Car();
        Observable<String> brandDeferObservable = car.brandDeferObservable();
        car.setBrand("五菱");
        brandDeferObservable.subscribe(getObserver());
    }

    private Observer<String> getObserver() {

        return new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("onSubscribe-----" + d.isDisposed());
            }

            @Override
            public void onNext(String s) {
                System.out.println("onNext------" + s);
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("onError-----" + e);
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete-----");
            }
        };

    }
}

public class Car {

    private String brand = "没有牌子";

    public void setBrand(String brand) {
        this.brand = brand;
    }
//   直接使用just会输出没有牌子
    public Observable<String> brandDeferObservable() {
//      return just(brand);
        return Observable.defer(new Callable<ObservableSource<? extends String>>() {
            @Override
            public ObservableSource<? extends String> call() throws Exception {
                return Observable.just(brand);
            }
        });
    }
//  达到同样的效果
//    public Observable<String> brandDeferObservable() {
//        return Observable.create(new ObservableOnSubscribe<String>() {
//            @Override
//            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
//                emitter.onNext(brand);
//            }
//        });
//    }
}


```
这是一个简单的例子，可以看到使用defer和不使用defer的区别。
```Java
/**
 * Returns an Observable that calls an ObservableSource factory to create an ObservableSource for each new Observer
 * that subscribes. That is, for each subscriber, the actual ObservableSource that subscriber observes is
 * determined by the factory function.
 * 
 * The defer Observer allows you to defer or delay emitting items from an ObservableSource until such time as an
 * Observer subscribes to the ObservableSource. This allows an {@link Observer} to easily obtain updates or a
 * refreshed version of the sequence.
 */
 
```
这是源码给出的关于defer的定义，大概意思是当发生订阅关系时才会创建Observable，这样做的好处是方便更新，让我们看一下源码：
```Java
//  defer是Observable的一个静态方法
public static <T> Observable<T> defer(Callable<? extends ObservableSource<? extends T>> supplier) {
    ObjectHelper.requireNonNull(supplier, "supplier is null");
    return RxJavaPlugins.onAssembly(new ObservableDefer<T>(supplier));
}
```
defer方法接收一个Callable作为参数，覆写call方法返回一个Observable

```Java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```
这是Callable接口。
```Java
public final class ObservableDefer<T> extends Observable<T> {
    final Callable<? extends ObservableSource<? extends T>> supplier;
    public ObservableDefer(Callable<? extends ObservableSource<? extends T>> supplier) {
        this.supplier = supplier;
    }
    @Override
    public void subscribeActual(Observer<? super T> s) {
        ObservableSource<? extends T> pub;
        try {
            pub = ObjectHelper.requireNonNull(supplier.call(), "null ObservableSource supplied");
        } catch (Throwable t) {
            Exceptions.throwIfFatal(t);
            EmptyDisposable.error(t, s);
            return;
        }
        pub.subscribe(s);
    }
}
```
这是defer对应的类。再根据示例来看，当调用brandDeferObservable方法时会得到ObservableDefer，当注册发生时，首先会调用Observable类中的subscribe方法，在subscribe方法中又会调用到ObservableDefer类中覆写的subscribeActual方法。在这个方法中可以看到，调用supplier的call()方法会将内部的ObservableSource实例化，之后pub再调用subscribe()方法，与observer达成注册关系。