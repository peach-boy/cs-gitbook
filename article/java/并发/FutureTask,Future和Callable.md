## FutureTask,Future和Callable


### 如何获取线程的执行结果
Java 通过 ThreadPoolExecutor 提供的 3 个 submit() 方法，两个invoke方法和 1 个 FutureTask 工具类来支持获得任务执行结果的需求
```
    // 提交Runnable任务
    Future<?> submit(Runnable task);
    // 提交Callable任务
    <T> Future<T> submit(Callable<T> task);
    // 提交Runnable任务及结果引用  
    <T> Future<T> submit(Runnable task, T result);
     // 批量提交任务
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)throws InterruptedException;
    // 批量提交任务（支持超时机制）
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,long timeout, TimeUnit unit)throws  InterruptedException;
```

#### 1.Future接口：
```
    //取消任务的方法
    boolean cancel(boolean mayInterruptIfRunning);
    //判断任务是否已取消的方法
    boolean isCancelled();
    //判断任务是否已结束的方法
    boolean isDone();
    //获得任务执行结果
    V get() throws InterruptedException, ExecutionException;
    //获得任务执行结果(支持超时机制`)
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```
1. get方法：`V get() throws InterruptedException, ExecutionException;`
    - 当FutureTask处于未启动或已启动状态时，执行FutureTask.get()方法将导致调用线程阻塞；
    -  当FutureTask处于已完成状态，调用FutureTask.get()方法将导致调用线程立即返回结果或者抛出异常。
 2. cancel方法: `boolean cancel(boolean mayInterruptIfRunning);`
    - 当FutureTask处于未启动状态时，执行FutureTask.cancel()方法将导致此任务永远不会被执行；
    - 当FutureTask处于已启动状态时，执行FutureTask.cancel（true）方法将以中断执行此任务线程的方式来试图停止任务；
    - 当FutureTask处于已启动状态时，执行FutureTask.cancel（false）方法将不会对正在执行此任务的线程产生影响（让正在执行的任务运行完成）；
    - 当FutureTask处于已完成状态时，执行FutureTask.cancel（…）方法将返回false。
3. get和cancel执行示例图
![FutureTask的get和cancel的执行示意图](/pic/java/FutureTask_1.png)

4. Callable+Future使用示例
 ```
 public class FutureTaskTest {
    static class CallableTask implements Callable<String> {
        private Integer sleepTime;

        public CallableTask(Integer sleepTime) {
            this.sleepTime = sleepTime;
        }

        @Override
        public String call() throws Exception {
            System.out.println(Thread.currentThread().getName()+"--begin--"+System.currentTimeMillis());
            Thread.sleep(sleepTime);
            System.out.println(Thread.currentThread().getName()+"-------end--"+System.currentTimeMillis());
            return Thread.currentThread().getName();
        }
    }

    public static void main(String[] args) {

        ExecutorService executorService = Executors.newFixedThreadPool(3);

        List<Callable<String>> callableList = new ArrayList<>();
        callableList.add(new CallableTask(2000));
        callableList.add(new CallableTask(5000));
        callableList.add(new CallableTask(2000));

        try {
            List<Future<String>> futures = executorService.invokeAll(callableList);
            System.out.println("return future--"+System.currentTimeMillis());
            for (Future future : futures) {
                System.out.println(future.get() + "--get---"+System.currentTimeMillis());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println("main thread compelete");
        executorService.shutdown();
    }
}
console result:
pool-1-thread-1--begin--1561992318002
pool-1-thread-3--begin--1561992318002
pool-1-thread-2--begin--1561992318002
pool-1-thread-3-------end--1561992320003
pool-1-thread-1-------end--1561992320003
pool-1-thread-2-------end--1561992323003
return future--1561992323003
pool-1-thread-1--get---1561992323003
pool-1-thread-2--get---1561992323004
pool-1-thread-3--get---1561992323004
 ```


#### 2.FutureTask工具类
FutureTask 是一个实实在在的工具类，这个工具类有两个构造函数，它们的参数和前面介绍的 submit() 方法类似。
```
FutureTask(Callable<V> callable);
FutureTask(Runnable runnable, V result);
```
FutureTask 实现了 Runnable 和 Future 接口，由于实现了 Runnable 接口，所以可以将 FutureTask 对象作为任务提交给 ThreadPoolExecutor 去执行，也可以直接被 Thread 执行；又因为实现了 Future 接口，所以也能用来获得任务的执行结果

 Callable+FutureTask使用示例
 ```
public static class CallableThread implements Callable<String>
{
    public String call() throws Exception
    {
        System.out.println("进入CallableThread的call()方法, 开始睡觉, 睡觉时间为" + System.currentTimeMillis());
        Thread.sleep(10000);
        return "123";
    }
}
    
public static void main(String[] args) throws Exception
{
    ExecutorService es = Executors.newCachedThreadPool();
    CallableThread ct = new CallableThread();
    FutureTask<String> f = new FutureTask<String>(ct);
    es.submit(f);
    es.shutdown();
        
    Thread.sleep(5000);
    System.out.println("主线程等待5秒, 当前时间为" + System.currentTimeMillis());
        
    String str = f.get();
    System.out.println("Future已拿到数据, str = " + str + ", 当前时间为" + System.currentTimeMillis());
}
 ```
 
 #### 3.InterruptedException如何处理
- 何时会发生：当阻塞方法收到中断请求的时候就会抛出InterruptedException异常
- 不要不管不顾:捕获到InterruptedException异常后恢复中断状态
```
public class TaskRunner implements Runnable {
    private BlockingQueue<Task> queue;
 
    public TaskRunner(BlockingQueue<Task> queue) { 
        this.queue = queue; 
    }
 
    public void run() { 
        try {
             while (true) {
                 Task task = queue.take(10, TimeUnit.SECONDS);
                 task.execute();
             }
         }
         catch (InterruptedException e) { 
             // Restore the interrupted status
             Thread.currentThread().interrupt();
         }
    }
}

```


 
