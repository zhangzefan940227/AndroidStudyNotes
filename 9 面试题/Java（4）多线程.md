### 1. <span id="java_thread_1">开启线程的三种方式？</span>

#### 继承Thread类创建线程类

1.  定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run方法称为执行体。
2.  创建Thread子类的实例，即创建了线程对象。
3.  调用线程对象的start方法来启动该线程。

```java
public class FirstThreadTest extends Thread {
    int i = 0;

    //重写run方法，run方法的方法体就是现场执行体
    public void run() {
        for (; i < 100; i++) {
            System.out.println(getName() + "  " + i);

        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "  : " + i);
            if (i == 20) {
                new FirstThreadTest().start();
                new FirstThreadTest().start();
            }
        }
    }

}
```

#### 通过Runnable接口创建线程类

1. 定义runnable接口的实现类，并重写该接口的run方法，该run方法的方法体同样是该线程的线程执行体。
2. 创建Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。
3. 调用线程对象的start方法来开启该线程。

```java
public class RunnableThreadTest implements Runnable {
    private int i;

    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 20) {
                RunnableThreadTest rtt = new RunnableThreadTest();
                new Thread(rtt, "新线程1").start();
                new Thread(rtt, "新线程2").start();
            }
        }

    }

}
```

#### 通过Callable和Future创建线程

1. 创建Callable接口的实现类，并实现call方法，该call方法将作为线程执行体，并且有返回值。
2. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call方法的返回值。
3. 使用FutureTask对象作为Thread对象的target创建并启动新线程。
4. 调用FutureTask对象的get方法来获取子线程执行结束后的返回值，调用get方法会阻塞线程。

```java
public class CallableThreadTest implements Callable<Integer> {

    public static void main(String[] args) {
        CallableThreadTest ctt = new CallableThreadTest();
        FutureTask<Integer> ft = new FutureTask<>(ctt);
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " 的循环变量i的值" + i);
            if (i == 20) {
                new Thread(ft, "有返回值的线程").start();
            }
        }
        try {
            System.out.println("子线程的返回值：" + ft.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Integer call() throws Exception {
        int i = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        return i;
    }
}
```

#### 创建线程的三种方式的对比

**采用实现Runnable、Callable接口的方式创建多线程时：**

优势：

	线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好的体现面向对象的思想。

劣势：

	稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

**使用继承Thread类的方式创建多线程时：**

优势：

	编写简单，如果需要访问当前线程，则不需使用Thread.currentThread方法，直接使用this即可获得当前线程。

劣势：

	线程类已经继承了Thread类，所以不能在继承其他父类。

### 2. <span id="java_thread_2">线程和进程的区别？</span>



### 3. <span id="java_thread_3">为什么要有线程，而不是仅仅用进程？</span>

	线程增加进程的并发度，线程能更有效的利用多处理器和多内核。

### 4. <span id="java_thread_4">run() 和 start() 方法的区别？</span>

	主要区别在于当程序调用start方法一个新线程将会被创建，并且在run方法中的代码将会在新线程上运行，然而在你直接调用run方法的时候，程序并不会创建新线程，run方法内部的代码将在当前线程上运行。
	
	另外一个区别在于，一旦一个线程被启动，你不能重复调用该Thread对象的start方法，调用已经启动线程的start方法将会报IIIegalStateException异常，而重复调用run方法是没有问题的。

### 5. <span id="java_thread_5">如何控制某个方法允许并发访问线程的个数？</span>

```java
Semaphore semaphore = new Semaphore(5);//线程run中只有5个线程可并发访问
semaphore.acquire();//申请一个信号
semaphore.release();//释放一个信号
```

### 6. <span id="java_thread_6">在 Java 中 wait() 和 sleep() 方法的区别</span>



### 7. <span id="java_thread_7">谈谈 wait/notify 关键字的理解</span>

	都是本地final方法，调用之前持有对象锁。
	
	wait，线程进入挂起状态，释放对象锁。
	
	notify，通知唤醒wait队列中的线程（某个）

### 8. <span id="java_thread_8">什么导致线程阻塞？</span>

	Thread.sleep t.join 等待输入

### 9. <span id="java_thread_9">线程如何关闭？</span>



### 10. <span id="java_thread_10">讲一下 Java 中的同步的方法</span>



### 11. <span id="java_thread_11">数据一致性如何保证？</span>



### 12. <span id="java_thread_12">如何保证线程安全？</span>



### 13.<span id="java_thread_13"> 如何实现线程同步？</span>



### 14. <span id="java_thread_14">两个进程同时要求读写，能不能实现？如何防止进程同步？</span>



### 15. <span id="java_thread_15">线程间操作 List</span>



### 16. <span id="java_thread_16">Java 中对象的生命周期</span>



### 17. <span id="java_thread_17">说说线程池</span>

**线程池的好处**

1. 线程池的重用

   线程的创建和销毁的开销是巨大的，而通过线程池的重用大大减少了这些不必要的开销，当然既然少了那么多消耗内存的开销，其线程执行速度也是突飞猛进的提升。

2. 控制线程池的并发数

   控制线程池的并发数可以有效地避免大量的线程争夺CPU资源而造成阻塞。

3. 线程池可以对线程进行管理

   线程池可以提供定时、定期、单线程、并发数控制等功能。

**线程池的详解**

1. ThreadPoolExecutor

   ```java
   
   public ThreadPoolExecutor(int corePoolSize,  
                                 int maximumPoolSize,  
                                 long keepAliveTime,  
                                 TimeUnit unit,  
                                 BlockingQueue<Runnable> workQueue,  
                                 ThreadFactory threadFactory,  
                                 RejectedExecutionHandler handler)
   ```

   这里是七个参数（更多的是用五个参数的构造方法）。

   **corePoolSize**：线程池中核心线程的数量。

   **maxinumPoolSize**：线程池中最大的线程数量。

   **keepAliveTime**：非核心线程的超时时长，当系统中非核心线程闲置时间超过keepAliveTime之后，则会被回收。如果ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，则该参数也表示核心线程的超时时长。

   **unit**：第三个参数的单位，有毫秒、秒、分等等。

   **workQueue**：线程池中的任务队列，该队列主要用来存储已经被提交但尚未执行的任务。存储在这里的任务是由ThreadPoolExecutor的execute方法提交来的。

   threadFactory：为线程池提供创建新线程的功能，这个我们一般使用默认即可。

   **handler**：拒绝策略，当线程无法执行新任务（一般是由于线程池中的线程数量已经达到最大数或者线程池关闭导致的）。默认情况下，当线程池无法处理新线程时，会抛出一个RejectedExecutionException。

   有以下：

   * 当currentSize < corePoolSize 时，直接启动一个核心线程并执行任务
   * 当currentSize >= corePoolSize，并且 workQueue 未满时，添加进来的任务会被安排到workQueue中等待执行
   * 当workQueue已满，但是currentSize < maxinumPoolSize 时，会立即开启一个非核心线程来执行任务
   * 当 currentSize >= corePoolSize、workQueue已满，并且currentSize > maxinumPoolSize 时，调用handler默认会抛出异常

2. 其他线程池

   1. FixedThreadPool

      有固定数量线程的线程池，其中corePoolSize = maxinumPoolSize，且keepAliveTime为0，适合线程稳定的场所。

   2. SingleThreadPool

      corePoolSize = maxinumPoolSzie = 1 且 keepAliveTime 为0，适合线程同步操作的场所。

   3. CachedThreadPool

      corePoolSize = 0，maximunPoolSize = Integer.MAX_VALUE（2的32次方-1）。

   4. ScheduledThreadPool

      是一个具有定时定期执行任务功能的线程池。

### <span id="java_thread_18">18. 并发编程面临的挑战</span>

1. 上下文切换
2. 死锁
3. 资源限制

参考：[并发编程面临的挑战](https://www.jianshu.com/p/f3e69850addf)

