---
layout:     post
title:      RxJava
subtitle:   RxJavaPlugins
date:       2018-10-27
author:     FS

catalog: true
tags:
    - RxJava

---
# RxJavaPlugins


```Java
public final class RxJavaPlugins{
    // 代码...
}
```
> Utility class to inject handlers to certain standard RxJava operations.

这是一个final类，官方的解释是，这是一个工具类用于向标某些标准的RxJava操作中注入处理程序。这个要怎么理解呢，接着看，

```Java
    /**
     * Calls the associated hook function.
     * @param <T> the value type
     * @param source the hook's input value
     * @return the value returned by the hook
     */

public static <T> Observable<T> onAssembly(@NonNull Observable<T> source) {
    Function<? super Observable, ? extends Observable> f = onObservableAssembly;
    if (f != null) {
        return apply(f, source);
    }
    return source;
}

static volatile Function<? super Observable, ? extends Observable> onObservableAssembly;

```
这个方法名翻译过来是装配的意思，从源码可以看出Function并没有被实例化，所以一般情况下，传入的Observable会被返回，但是也有特殊情况，
```Java
    /**
     * Sets the specific hook function.
     * @param onObservableAssembly the hook function to set, null allowed
     */
    @SuppressWarnings("rawtypes")
    public static void setOnObservableAssembly(@Nullable Function<? super Observable, ? extends Observable> onObservableAssembly) {
        if (lockdown) {
            throw new IllegalStateException("Plugins can't be changed anymore");
        }
        RxJavaPlugins.onObservableAssembly = onObservableAssembly;
    }


```
这个方法用来实例化onObservableAssembly，

```Java
    @NonNull
    static <T, R> R apply(@NonNull Function<T, R> f, @NonNull T t) {
        try {
            return f.apply(t);
        } catch (Throwable ex) {
            throw ExceptionHelper.wrapOrThrow(ex);
        }
    }

public interface Function<T, R> {
    /**
     * Apply some calculation to the input value and return some other value.
     * @param t the input value
     * @return the output value
     * @throws Exception on error
     */
    R apply(@NonNull T t) throws Exception;
}


```
这个Function接口是每次操作Observable时会调用apply方法，这就是钩子函数。


```Java
// 这是Observable抽象类中唯一的抽象方法
protected abstract void subscribeActual(Observer<? super T> observer);
```
