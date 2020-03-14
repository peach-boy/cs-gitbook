### int和Integer区别及==与equal区别

#### 包装类型
基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成，这是编译器自动进行的。
```
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

#### 缓冲池
new Integer(123) 与 Integer.valueOf(123) 的区别在于：
- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容,在 Java 8 中，Integer 缓存池的大小默认为 -128~127。
```
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

#### ==和equal()区别

- 基础数据类型：==判断两个值是否相等，基础类没有eqauls()方法
- 引用类型：==判断的是连个变量是否引用同一个对象，而equals()用于判断两个变量引用的对象是否等价，这需要类自己实现equals()方法。equals()是顶级类object()中的方法，默认实现等价于==，比较的是对象内存地址，但是实际业务大多数并不需要比较内存地址，所以就需要类自己实现equal()方法，object的默认实现如下：
```
   public boolean equals(Object var1) {
        return this == var1; //也是等价于==
    }
```