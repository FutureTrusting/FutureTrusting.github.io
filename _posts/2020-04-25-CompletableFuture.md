---
layout:     post
title:      CompletableFuture
subtitle:   CompletableFuture
date:       2020-04-25
author:     Square
header-img: img/wallhaven-zm3mxw.jpg
catalog: true
tags:
    - Java8
---

### 1. runAsync 和 supplyAsync方法
CompletableFuture 提供了四个静态方法来创建一个异步操作。
```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```
没有指定Executor的方法会使用ForkJoinPool.commonPool() 作为它的线程池执行异步代码。    
如果指定线程池，则使用指定的线程池运行。以下所有的方法都类同。    
- runAsync方法不支持返回值。
- supplyAsync可以支持返回值。

<Strong>示例</Strong>
```java
        //无返回值
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                System.err.println(e.getMessage());
            }
            System.out.println("run end ...");
        });
        future.join();
        //有返回值
        CompletableFuture<Long> future2 = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                System.err.println(e.getMessage());
            }
            System.out.println("run end ...");
            return System.currentTimeMillis();
        });
        long time = future2.join();
        System.out.println("time = " + time);
```
### 2. 计算结果完成时的回调方法
当CompletableFuture的计算结果完成，或者抛出异常的时候，可以执行特定的Action。主要是下面的方法：
```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```
可以看到Action的类型是BiConsumer<? super T,? super Throwable>它可以处理正常的计算结果，或者异常情况。
- whenComplete 和 whenCompleteAsync 的区别：
- whenComplete：是执行当前任务的线程执行继续执行 whenComplete 的任务。
- whenCompleteAsync：是执行把 whenCompleteAsync 这个任务继续提交给线程池来进行执行。

<Strong>示例</Strong>
```java
      CompletableFuture<Long> future1 = CompletableFuture.supplyAsync(() -> {
              try {
                  TimeUnit.SECONDS.sleep(1);
              } catch (InterruptedException e) {
                  System.err.println(e.getMessage());
              }
              System.out.println("run end ...");
              return System.currentTimeMillis();
          }).whenComplete((t, action) -> System.err.println("执行完成\n" + "t=" + t +"\n"+ "action=" + action));
      long time = future1.join();
      System.out.println("time = " + time);
      CompletableFuture<Long> future2 = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                System.err.println(e.getMessage());
            }
            int i=10/0;
            System.out.println("run end ...");
            return System.currentTimeMillis();
        }).exceptionally(t -> 1L);
        long time = future2.join();
        System.out.println("time = " + time);
```
### 3. thenApply 方法
当一个线程依赖另一个线程时，可以使用 thenApply 方法来把这两个线程串行化。
```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```
- Function<? super T,? extends U>
- T：上一个任务返回结果的类型
- U：当前任务的返回值类型

<Strong>示例</Strong>
```java
        CompletableFuture<Long> future = CompletableFuture.supplyAsync(() -> {
            long result = new Random().nextInt(10);
            System.out.println("result1=" + result);
            return result;
        }).thenApply(t -> {
            long result = t * 10;
            System.out.println("result2=" + result);
            return result;
        });
        long result = future.join();
        System.out.println(result);
```
第二个任务依赖第一个任务的结果。
### 4. handle 方法
handle 是执行任务完成时对结果的处理。
handle 方法和 thenApply 方法处理方式基本一样。不同的是 handle 是在任务完成后再执行，还可以处理异常的任务。   
thenApply 只可以执行正常的任务，任务出现异常则不执行 thenApply 方法。   
```java
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn,Executor executor);
```

<Strong>示例</Strong>
```java
  CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
                int i= 10/0;
                return new Random().nextInt(10);
            }).handle((param, throwable) -> {
                int result = -1;
                if(throwable==null){
                    result = param * 2;
                }else{
                    System.out.println(throwable.getMessage());
                }
                return result;
            });
            System.out.println(future.join());
```
从示例中可以看出，在 handle 中可以根据任务是否有异常来进行做相应的后续处理操作。
而 thenApply 方法，如果上个任务出现错误，则不会执行 thenApply 方法。

### 5. thenAccept 消费处理结果
接收任务的处理结果，并消费处理，无返回结果。
```java
public CompletionStage<Void> thenAccept(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor);
```
<Strong>示例</Strong>
```java
        CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
           return new Random().nextInt(10);
        }).thenAccept(integer -> {
            System.out.println(integer);
        });
        future.join();
```
从示例代码中可以看出，该方法只是消费执行完成的任务，并可以根据上面的任务返回的结果进行处理。并没有后续的输错操作。

### 6. thenRun 方法
跟 thenAccept 方法不一样的是，不关心任务的处理结果。  
只要上面的任务执行完成，就开始执行 thenAccept 。   
```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```
<Strong>示例</Strong>
```java
        CompletableFuture<Void> future = CompletableFuture.supplyAsync(() ->{
            return new Random().nextInt(10);
        }).thenRun(() -> {
            System.out.println("thenRun ...");
        });
        future.join();
```
该方法同 thenAccept 方法类似。不同的是上个任务处理完成后，并不会把计算的结果传给 thenRun 方法。   
只是处理玩任务后，执行 thenAccept 的后续操作。  

### 7. thenCombine 合并任务
thenCombine 会把 两个 CompletionStage 的任务都执行完成后，把两个任务的结果一块交给 thenCombine 来处理。
```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn,Executor executor);
```
<Strong>示例</Strong>
```java
   CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "hello");
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "world");
        CompletableFuture<String> result = future1.thenCombine(future2, (t, u) -> {
            System.err.println("t=" + t + "\n" + "u=" + u);
            return t + u;
        });
        System.out.println(result.join());
```
### 8. thenAcceptBoth
当两个CompletionStage都执行完成后，把结果一块交给thenAcceptBoth来进行消耗
```java
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action,     Executor executor);
```
<Strong>示例</Strong>
```java
 CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1=" + t);
            return t;
        });
        CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2=" + t);
            return t;
        });
        final CompletableFuture<Void> acceptBoth = f1.thenAcceptBoth(f2, (t, u) -> {
            System.out.println("f1=" + t + ";f2=" + u + ";");
        });
```
### 9. applyToEither 方法
两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的转化操作。
```java
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn,Executor executor);
```
<Strong>示例</Strong>
```java
   CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                try {
                    TimeUnit.SECONDS.sleep(t);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.err.println("f1="+t);
                return t;
            });
            CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                try {
                    TimeUnit.SECONDS.sleep(t);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.err.println("f2="+t);
                return t;
            });
            CompletableFuture<Integer> result = f1.applyToEither(f2, t -> {
                System.err.println(t);
                return t * 2;
            });
            System.err.println(result.join());
```
### 10. acceptEither 方法
两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的消耗操作。
```java
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action,Executor executor);
```
<Strong>示例</Strong>
```java
             CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                try {
                    TimeUnit.SECONDS.sleep(t);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("f1="+t);
                return t;
            });
            CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                try {
                    TimeUnit.SECONDS.sleep(t);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("f2="+t);
                return t;
            });
            f1.acceptEither(f2, t -> {
                System.out.println(t);
            });
```
### 11. runAfterEither 方法
两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable）
```java
public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor);
```
<Strong>示例</Strong>
```java
            CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                try {
                    TimeUnit.SECONDS.sleep(t);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("f1="+t);
                return t;
            });
            CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                try {
                    TimeUnit.SECONDS.sleep(t);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("f2="+t);
                return t;
            });
            f1.runAfterBoth(f2, () ->{
                System.out.println("上面两个任务任意一个执行完成了。");
            });
```
### 12. runAfterBoth 方法
两个CompletionStage，都完成了计算才会执行下一步的操作（Runnable）
```java
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor);
```
<Strong>示例</Strong>
```java
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> {
                    int t = new Random().nextInt(3);
                    try {
                        TimeUnit.SECONDS.sleep(t);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("f1="+t);
                    return t;
                });
                CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(() -> {
                    int t = new Random().nextInt(3);
                    try {
                        TimeUnit.SECONDS.sleep(t);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("f2="+t);
                    return t;
                });
                f1.runAfterBoth(f2, () ->{
                    System.out.println("上面两个任务都执行完成了。");
                });
```
### 13. thenCompose 方法
thenCompose 方法允许你对两个 CompletionStage 进行流水线操作，第一个操作完成时，将其结果作为参数传递给第二个操作。
```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn) ;
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor) ;
```
<Strong>示例</Strong>
```java
            CompletableFuture<Integer> f = CompletableFuture.supplyAsync(() -> {
                int t = new Random().nextInt(3);
                System.out.println("t1="+t);
                return t;
            }).thenCompose(param -> CompletableFuture.supplyAsync(() -> {
                int t = param * 10;
                System.out.println("t2="+ t);
                return t;
            }));
            System.out.println("thenCompose result : "+f.join());
```
### 14. CompletableFutureUtil
```java
        //任务1：洗水壶->烧开水
        CompletableFuture<String> f1 =
                CompletableFuture.supplyAsync(()->{
                    System.out.println("T1:洗水壶...");
                    sleep(1, TimeUnit.SECONDS);

                    System.out.println("T1:烧开水...");
                    sleep(15, TimeUnit.SECONDS);
                    return "白开水";
                });
        //任务2：洗茶壶->洗茶杯->拿茶叶
        CompletableFuture<String> f2 =
                CompletableFuture.supplyAsync(()->{
                    System.out.println("T2:洗茶壶...");
                    sleep(1, TimeUnit.SECONDS);

                    System.out.println("T2:洗茶杯...");
                    sleep(2, TimeUnit.SECONDS);

                    System.out.println("T2:拿茶叶...");
                    sleep(1, TimeUnit.SECONDS);
                    return "龙井";
                });
        //任务3：任务1和任务2完成后执行：泡茶
        CompletableFuture<String> f3 =
                f1.thenCombine(f2,(f11, f22)->{
                    System.out.println("T1:拿到水:" + f11);
                    System.out.println("T1:泡茶..."+f22);
                    return "上茶:" + f11;
                });
        //等待任务3执行结果
        System.out.println(f3.join());
```
```java
        final ForkJoinPool forkJoinPool = new ForkJoinPool(2 * (Runtime.getRuntime().availableProcessors()) + 1);
        CompletableFuture<DeliverTmGisDTO> future1 = CompletableFuture.supplyAsync(() -> new DeliverTmGisDTO(null, commonGisService.gisAddrSlip(request.getSrcAddress()), null))
                .exceptionally(x -> null);
        CompletableFuture<DeliverTmGisDTO> future2 = CompletableFuture.supplyAsync(() -> new DeliverTmGisDTO(null, null, commonGisService.gisAddrSlip(request.getDestAddress())))
                .exceptionally(y -> null);
        CompletableFuture<DeliverTmGisDTO> future3 = CompletableFuture.supplyAsync(() -> new DeliverTmGisDTO(commonOrgService.depthGetClientCodeAndCheckCode(orgId, 0), null, null), forkJoinPool)
                .exceptionally(z -> null);
        final List<CompletableFuture<DeliverTmGisDTO>> futures = Arrays.asList(future1, future2, future3);
        List<DeliverTmGisDTO> deliverTmGis;
        try {
            deliverTmGis = CommonCompletableFutureUtil.sequenceExceptionallyCompleteMeetFirstfFailure(futures).join();
        } catch (Exception e) {
            log.error("BspOrderController.deliverTmV3 error:{}", e.getMessage());
            throw e;
        } finally {
            log.info("forkJoinPool.shutdownNow success");
            forkJoinPool.shutdownNow();
        }
```
```java
package com.sf.united.store.common.utils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import static java.util.stream.Collectors.toList;
/**
 * created by liurenjin on 2019/2/21
 * 参考
 * https://www.cnblogs.com/dennyzhangdd/p/7010972.html
 */
public class CommonCompletableFutureUtil {
    private static final Logger log = LoggerFactory.getLogger(CommonCompletableFutureUtil.class);
    /*组合多个CompletableFuture为一个CompletableFuture,所有子任务全部完成，组合后的任务才会完成。带返回值，可直接get.
     *
     * */
    public static <T> CompletableFuture<List<T>> sequence(List<CompletableFuture<T>> futuresList) {
        log.info("CompletableFuture sequence start ");
        //1.构造一个空CompletableFuture，子任务数为入参任务list size
        //2.流式（总任务完成后，每个子任务join取结果，后转换为list）
        return CompletableFuture.allOf(futuresList.toArray(new CompletableFuture<?>[0]))
                .thenApply(v -> futuresList.stream()
                        .map(CompletableFuture::join)
                        .collect(toList())
                );
    }
    //任何一个任务 异常
    public static <T> CompletableFuture<List<T>> sequenceExceptionallyCompleteMeetFirstfFailure(List<CompletableFuture<T>> futuresList) {
        log.info("CompletableFuture sequenceExceptionallyCompleteMeetFirstfFailure start ");
        CompletableFuture<List<T>> result = CompletableFuture.allOf(futuresList.toArray(new CompletableFuture<?>[0]))
                .thenApply(v -> futuresList.stream()
                        .map(CompletableFuture::join)
                        .collect(toList())
                );
        futuresList.forEach(f -> f.whenComplete((t, ex) -> {
            if (ex != null) {
                log.error("CompletableFuture  exception " + ex.getMessage(), ex);
                result.completeExceptionally(ex);
            }
        }));
        log.info("CompletableFuture sequenceExceptionallyCompleteMeetFirstfFailure end ");
        return result;
    }
}
```










