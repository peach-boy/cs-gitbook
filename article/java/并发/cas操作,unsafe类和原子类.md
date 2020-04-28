## cas操作,unsafe类和原子类

### CAS操作
cas是一种乐观锁的实现方式，在juc包中有大量的应用，如：AtomicInteger

```
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();

public final int incrementAndGet() {
        return U.getAndAddInt(this, VALUE, 1) + 1;
}
```
其中我们可以看到最终的cas操作是交给unsafe来实现的。

### unsafe类

### 原子类
