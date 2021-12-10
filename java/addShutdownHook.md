# addShutdownHook 方法
## 方法
```java
public void addShutdownHook(Thread hook) {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        sm.checkPermission(new RuntimePermission("shutdownHooks"));
    }
    ApplicationShutdownHooks.add(hook);
}
```
```java
Runtime.getRuntime().addShutdownHook(new Thread())
```
## 作用
在 JVM 中增加一个关闭的钩子，当 JVM 关闭的时候，会执行系统中已经设置的所有通过 addShutdownHook 添加的钩子，当系统执行完这些钩子后，JVM 才会关闭。
## Tips
Linux 服务器上使用 `kill pid` 可以被 Runtime 捕获，`kill -9 pid` 不能被捕获