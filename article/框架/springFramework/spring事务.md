## spring事务


### 1.事务隔离级别：是指若干个并发的事务之间的隔离程度

在一个典型的应用程序中，多个事务同时运行，经常会为了完成他们的工作而操作同一个数据。并发虽然是必需的，但是会导致以下问题：
* 脏读(Dirty read):脏读发生在一个事务读取了被另一个事务改写但尚未提交的数据时。如果这些改变在稍后被回滚了，那么第一个事务读取的数据就会是无效的.
* 不可重复读(Nonrepeatable read):不可重复读发生在一个事务执行相同的查询两次或两次以上，但每次查询结果都不相同时。这通常是由于另一个并发事务在两次查询之间更新了数据.
* 幻读(Phantom reads):幻读和不可重复读相似。当一个事务（T1）读取几行记录后，另一个并发事务（T2）插入了一些记录时，幻读就发生了。在后来的查询中，第一个事务（T1）就会发现一些原来没有的额外记录.

TransactionDefinition 接口中定义了五个表示隔离级别的常量
|名称|是否默认|说明|
|-|-|-|
|DEFAULT|是|表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是ISOLATION_READ_COMMITTED|
|READ_UNCOMMITTED|否|该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。|
|READ_COMMITTED|否|该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值|
|REPEATABLE_READ|否|该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。|
|SERIALIZABLE|否|所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别|
 

### 2.事务传播行为（为了解决业务层方法之间互相调用的事务问题）
所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：
|名称|是否默认|说明|
|-|-|-|
|REQUIRED     |是|如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。|
|SUPPORT      |否|如果当前存在事务，则加入该事务；如果当前没有事务，则以非事物方式运行。|
|MANDATORY    |否|如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。|
|REQUIRES_NEW |否|创建一个新的事务，如果当前存在事务，则把当前事务挂起|
|NOT_SUPPORT  |否|以非事务方式运行，如果当前存在事务，则把当前事务挂起|
|NEVER        |否|以非事务方式运行，如果当前存在事务，则抛出异常|
|NESTED       |否|如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于REQUIRED|

### 3.事务超时
所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒.
>@Transactional(timeout=30)


### 4.事务的只读属性
如果一个事务只对数据库执行读操作，那么该数据库就可能利用那个事务的只读特性，采取某些优化措施。通过把一个事务声明为只读，可以给后端数据库一个机会来应用那些它认为合适的优化措施。由于只读的优化措施是在一个事务启动时由后端数据库实施的， 因此，只有对于那些具有可能启动一个新事务的传播行为（PROPAGATIONREQUIRESNEW、PROPAGATIONREQUIRED、 ROPAGATIONNESTED）的方法来说，将事务声明为只读才有意义。
>@Transactional(readOnly=true)
该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。

### 5.回滚规则
在默认设置下，事务只在出现运行时异常（runtime exception）时回滚，而在出现受检查异常（checked exception）时不回滚（这一行为和EJB中的回滚行为是一致的）。
不过，可以声明在出现特定受检查异常时像运行时异常一样回滚。同样，也可以声明一个事务在出现特定的异常时不回滚，即使特定的异常是运行时异常
- 指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)
- 指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class})


### 6.@Transactional注解属性说明
以下是@Transactional源码：
```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}

```
@Transactional属性说明
|属性名|说明|默认行为|
|-|-|-|
|propagation|事务传播行为|Propagation.REQUIRED|
|isolation|事务隔离级别|Isolation.DEFAULT|
|readOnly|事务读写性|false|
|timeout|超时时间，单位秒|-1，即无超时时间|
|rollbackFor|一组异常类，遇到时进行回滚|默认为{}，即所有运行时异常都会滚|
|rollbackForClassName|一组异常类名，遇到时进行回滚|默认{}，即所有运行时异常都会滚|
|noRollbackFor|一组异常类，遇到时不回滚|默认为{}|
|noRollbackForClassName|一组异常类名，遇到时不回滚|默认{}|
