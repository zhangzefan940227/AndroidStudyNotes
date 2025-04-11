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

## Android 部分（一）基础知识点

### 1. <span id="android_base_1">四大组件是什么？</span>

四大组件：Activity、Service、BroadcastReceiver、ContentProvider。

### 2. <span id="android_base_2">四大组件的生命周期和简单用法</span>

### **Activity：**

典型的生命周期好像没什么可说的，主要说一下特殊情况下的生命周期。

1. 横竖屏的切换

   在横竖屏切换的过程中，会发生Activity被销毁并被重建的过程。

   在了解这种情况下的生命周期，首先应该了解这两个回调：onSaveInstance State 和 onRestoreInstance State。

   在Activity由于异常情况下终止时，系统会调用 onSaveInstanceState 来保存当前 Activity 的状态。这个方法的调用是在onStop之前，它和onPause没有既定的时序关系，该方法只有在Activity被异常终止的情况下调用。当异常终止的Activity被重建之后，系统会调用onRestoreInstanceState，并且把Activity销毁时onSaveInstanceState方法所保存的Bundle对象参数同时传递给onRestoreInstanceState和onCreate方法。因为，可以通过onRestoreInstanceState方法来恢复Activity的状态，该方法的调用时机是在onStart之后。其中，onCreate和onRestoreInstanceState方法来恢复Activity状态的区别：onRestoreInstanceState回调则表明其中Bundle对象非空，不用加非空判断，而onCreate需要非空判断，建议使用onRestoreInstanceState。

![](http://upload-images.jianshu.io/upload_images/3985563-23d90471fa7f12d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

	横竖屏切换的生命周期：onPause() --> onSaveInstanceState() --> onStop() --> onDestory() --> onCreate() --> onStart() --> onRestoreInstanceState() --> onResume() 
	
	可以通过在AndroidManifest文件的Activity中指定如下属性：
	
	android:configChanges = "orientation| screenSize"
	
	来避免横竖屏切换时，Activity的销毁和重建，而是回调了下面的方法：

```java
	@Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
```

2. 资源内存不足导致优先级低的Activity被杀死

   优先级：前台Activity > 可见但非前台Activity > 后台Activity

**启动模式：**

Android 提供了四种Activity启动方式：

标准模式：standard

栈顶复用模式：singleTop

栈内复用模式：singleTask

单例模式：singleInstance

1. 标准模式 standard

   每启动一次Activity，就会创建一个新的Activity实例并置于栈顶。谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。

   特殊情况下，如果在Service或Application中启动一个Activity，其并没有所谓的任务栈，可以使用标记位Flag来解决。解决办法：为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，创建一个新栈。

2. 栈顶复用模式 singleTop

   如果需要新建的Activity位于任务栈栈顶，那么此Activity的实例就不会重建，而是复用栈顶的实例。并回调：

   ```java
       @Override
       protected void onNewIntent(Intent intent) {
           super.onNewIntent(intent);
       }
   ```

   由于不会重建一个Activity实例，则不会回调其他生命周期方法。

   应用场景：在通知栏点击收到的通知，然后需要启动一个Activity，这个Activity就可以用singleTop，否则每次点击都会新建一个Activity。

3. 栈内复用模式 singleTask

   该模式是一种单例模式，即一个栈内只有一个该Activity实例。该模式，可以通过在AndroidManifest文件的Activity中指定该Activity需要加载到哪个栈中，即singleTask的Activity可以指定想要加载的目标栈。singleTask和taskAffinity配合使用，指定开启的Activity加入到哪个栈中。

   ```xml
   <activity android:name=".Activity1"
       android:launchMode="singleTask"
       android:taskAffinity="com.lvr.task"
       android:label="@string/app_name">
   </activity>
   ```

   关于taskAffinity的值：每个Activity都有taskAffinify属性，这个属性指出了它希望进入的Task。如果一个Activity没有显式的指明该Activity的taskAffinity，那么它的这个属性就等于Application指明的taskAffinity，如果Application也没有指明，那么该taskAffinity的值就等于包名。

   执行逻辑：

   在这种模式下，如果Activity指定的栈不存在，则创建一个栈，并把创建的Activity压入栈内。如果Activity指定的栈存在，如果其中没有该Activity实例，则会创建Activity并压入栈顶，如果其中有该Activity实例，则把该Activity实例之上的Activity杀死清除出战，重用并让该Activity实例处在栈顶，然后调用onNewIntent()方法。

   应用场景：

   在大多数App的主页，对于大部分应用，当我们在主界面点击返回按钮都是退出应用，那么当我们第一次进入主界面之后，主界面位于栈底，以后不管我们打开了多少个Activity，只要我们再次回到主界面，都应该使用将主界面Activity上所有的Activity移除的方式来让主界面Activity处于栈顶，而不是往栈顶新加一个主界面Activity的实例，通过这种方式能够保证退出应用时所有的Activity都能被销毁。

4. 单例模式 singleInstance

   作为栈内复用的加强版，打开该Activity时，直接创建一个新的任务栈，并创建该Activity实例放入栈中。一旦该模式的Activity实例已经存在于某个栈中，任何应用在激活该Activity时都会重用该栈中的实例。

   应用场景：呼叫来电界面

**特殊情况：前台栈和后台栈的交互**

![](http://upload-images.jianshu.io/upload_images/3985563-4aeb1947bba27e44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/3985563-f2eaf1005cdf1b1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调用SingleTask模式的后台任务栈的Activity，会把整个栈的Activity压入当前栈的栈顶。singleTask会具有clearTop特性，会把之上的栈内Activity清除。

**Activity的Flags：**

Activity的Flags很多，这里介绍集中常用的，用于设定Activity的启动模式，可以在启动Activity时，通过Intent.addFlags()方法设置。

1. FLAG_ACTIVITY_NEW_TASK 即 singleTask
2. FLAG_ACTIVITY_SINGLE_TOP 即 singleTop
3. FLAG_ACTIVITY_CLEAR_TOP 当他启动时，在同一个任务栈中所有位于它之上的Activity都要出栈。如果和singleTask模式一起出现，若被启动的Activity已经存在栈中，则清除其之上的Activity，并调用该Activity的onNewIntent方法。如果被启动的Activity采用standard模式，那么该Activity连同之上的所有Activity出栈，然后创建新的Activity实例并压入栈中。


**Activity的启动过程**

![](https://i.loli.net/2018/06/05/5b1642023c7d7.png)

**应用启动过程**

1. Launcher通过Binder进程间通信机制通知AMS，它要启动一个Activity
2. AMS通过Binder进程间通信机制通知Launcher进入Paused状态
3. Launcher通过Binder进程间通信机制通知AMS，它已经准备就绪进入Paused状态，于是AMS就创建一个新的线程，用来启动一个ActivityThread实例，即将要启动的Activity就是在这个ActivityThread实例中运行
4. ActivityThread通过Binder进程间通信机制将一个ApplicationThread类型的Binder对象传递给AMS，以便以后AMS能够通过这个Binder对象和它进行通信
5. AMS通过Binde进程间通信机制通知ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了


### Service

是Android中实现程序后台运行的解决方案，它非常适用于去执行那些不需要和用户交互而且还要长期运行的任务。Service默认并不会运行在子线程中，它也不运行在一个独立的进程中，它同样执行在UI线程中，因此，不要在Service中执行耗时的操作，因此，不要在Service中执行耗时的操作，除非你在Service中创建了子线程来完成耗时操作。
