# CompletableFuture
java.util.concurrent.Future 不便于获取结果，只能通过阻塞或轮询的方式获取，阻塞与异步编程的初衷相违背，轮询浪费 CPU 资源

Java 8 提供了 java.util.concurrent.CompletableFuture 来扩展 Future 的功能

## 主动完成计算
获取执行结果，join 抛出的是 Unchecked 异常，get 抛出的是经过检查的异常
```java
V get();
V get(long timeout,Timeout unit);
T getNow(T defaultValue);
T join();
```
在以下情况中，get() 的结果和 Future 中一样
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "Res";
});
try {
    System.out.println(future.get());
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```
可以加入以下代码完成计算或抛出异常
```java
completableFuture.complete("Res2");
completableFuture.completeExceptionally(new Exception());
```
## 创建 CompletableFuture 对象
没有指定 Executor 的情况下会使用 ForkJoinPool.commonPool() 作为线程池
```java
public static CompletableFuture<Void> 	runAsync(Runnable runnable)
public static CompletableFuture<Void> 	runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> 	supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> 	supplyAsync(Supplier<U> supplier, Executor executor)
```
创建计算完成的 CompletableFuture
```java
public static <U> CompletableFuture<U> completedFuture(U value)
```
## 计算结果完成时的处理
以 Async 结尾的将以异步方式执行

thenAccept: 当前任务正常完成以后执行，当前任务的执行结果可以作为下一任务的输入参数，无返回值

thnRun: 对不关心上一步的计算结果，执行下一个操作

thenApply: 当前任务正常完成以后执行，当前任务的执行的结果会作为下一任务的输入参数，有返回值

thenCompose 和 thenApply 的区别：

thenApply 的功能相当于将 CompletableFuture<T> 转换成 CompletableFuture\<U>，改变的是同一个 CompletableFuture 中的泛型类型，thenCompose 用来连接两个 CompletableFuture，返回值是一个新的 CompletableFuture
```java
public CompletableFuture<T> 	whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> 	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> 	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T>     exceptionally(Function<Throwable,? extends T> fn)

public CompletionStage<Void> 	thenAccept(Consumer<? super T> action);
public CompletionStage<Void> 	thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> 	thenAcceptAsync(Consumer<? super T> action,Executor executor);

public CompletionStage<Void> 	thenRun(Runnable action);
public CompletionStage<Void> 	thenRunAsync(Runnable action);
public CompletionStage<Void> 	thenRunAsync(Runnable action,Executor executor);

public <U> CompletableFuture<U>    	thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U>     thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U>     thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)

public CompletionStage<T> exceptionally(Function<Throwable, ? extends T> fn)
```
## 组合
结合两个CompletionStage的结果，进行转化后返回

thenCombine 是结合两个任务的返回值进行转化后再返回，那如果不需要返回就需要 thenAcceptBoth，同理，如果连两个任务的返回值也不关心就需要 runAfterBoth
```java
CompletableFuture<Integer> priceFuture = CompletableFuture.supplyAsync(() -> 100);
CompletableFuture<Double> discountFuture = CompletableFuture.supplyAsync(() -> 0.7);
CompletableFuture<Double> resFuture = priceFuture.thenCombineAsync(discountFuture, (price, discount) -> {
    System.out.println("price: " + price);
    System.out.println("discount: " + discount);
    return price * discount;
});
System.out.println("res: " + resFuture.join());
```
## Either
acceptEither 方法是当任意一个 CompletionStage 完成的时候，action这个消费者就会被执行，这个方法返回 CompletableFuture<Void>

applyToEither 方法是当任意一个 CompletionStage 完成的时候，fn会被执行，它的返回值会当作新的 CompletableFuture\<U> 的计算结果
```java
public CompletableFuture<Void> 	acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> 	acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
public CompletableFuture<Void> 	acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)
public <U> CompletableFuture<U> 	applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> 	applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)
public <U> CompletableFuture<U> 	applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn, Executor executor)
```
## allOf 和 anyOf
allOf: 当所有的 CompletableFuture 都执行完后执行计算

anyOf: 最快的那个 CompletableFuture 执行完之后执行计算
```java
public static CompletableFuture<Void>  allOf(CompletableFuture<?>... cfs)
public static CompletableFuture<Object>  anyOf(CompletableFuture<?>... cfs)
```