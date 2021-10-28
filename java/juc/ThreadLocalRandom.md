# ThreadLocalRandom
```java
ThreadLocalRandom random = ThreadLocalRandom.current();
int randomNum = random.nextInt(10, 100);
```

不要将ThreadLocalRandom的实例在不同线程中共享，会造成随机值相同的问题。
[ThreadLocalRandom 的正确用法](https://blog.csdn.net/sinat_27143551/article/details/106981621)

对比于 java.util.Random, java.util.concurrent.ThreadLocalRandom区别如下：
1. Random是线程安全的，理论上可以通过它同时在多个线程中获得互不相同的随机数，这样的线程安全是通过AtomicLong及synchronized实现的。Random使用AtomicLong CAS操作来更新它的seed，尽管在很多非阻塞式算法中使用了非阻塞式原语，CAS在资源高度竞争时的表现依然糟糕；
2. ThreadLocalRandom继承自Random，虽然可以使用ThreadLocal<Random>来避免竞争，但是无法避免synchronized/compareAndSet带来的开销。考虑到性能还是建议替换使用ThreadLocalRandom（有3倍以上提升），这不是ThreadLocal包装后的Random，而是真正的使用ThreadLocal机制重新实现的Random;
3. ThreadLocalRandom使用一个普通的long而不是使用Random中的AtomicLong作为seed；
4. 不能自己创建ThreadLocalRandom实例，因为它的构造函数是私有的，可以使用它的静态工厂ThreadLocalRandom.current()创建；
5. 它是CPU缓存感知式的，使用8个long虚拟域来填充64位L1高速缓存行