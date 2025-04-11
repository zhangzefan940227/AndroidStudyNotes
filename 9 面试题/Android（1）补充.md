
### 13. <span id="android_base_13"> Fragment详解</span>

#### 生命周期

![](https://images2015.cnblogs.com/blog/945877/201611/945877-20161123093212096-2032834078.png)

#### Fragment使用方式

**静态使用**

步骤：

1. 创建一个类继承Fragment，重写onCreateView方法，来确定Fragment要显示的布局
2. 在Activity的布局xml文件中添加< fragment >标签，并声明name属性为Fragemnt的全限定名，**而且必须给该控件(fragment)一个id或者tag**

**动态使用**

步骤：

1. 在Activity的布局xml文件中添加一个FrameLayout，此控件是Fragment的容器。
2. 准备好Fragment并实例化，获取Fragment管理器对象 FragmentManager。
3. 然后通过Fragment管理器调用beginTransaction()，实例化FragmentTransaction对象，即开启事务。
4. 执行事务并提交。FragmentTransaction对象有以下几种方法：
   * add()  向Activity中添加一个Fragment
   * remove() 从Activity中移除一个Fragment，如果被移除的Fragment没有添加到回退栈，这个Fragment实例将会被销毁
   * replace() 使用一个Fragment替换当前的，实际上就是remove + add
   * hide() 隐藏当前的Fragment，仅仅设为不可见，并不会销毁
   * show() 显示之前隐藏的Fragment
   * detach() 会将view从UI中移除，和remove()不同，此时fragment的状态依然由FragmentManager维护
   * attach() 重建View视图，附加到UI上并显示
   * commit() 提交事务

**回退栈**

Fragment的回退栈是用来保存每一次Fragment事务发生的变化，如果将Fragment任务添加到回退栈，当用户点击后退按钮时，将会看到上一次保存的Fragment，一旦Fragment完全从回退栈中弹出，用户再次点击后退键，则会退出当前Activity。

FragmentTransaction.addToBackStack(String) 把当前事务的变化情况添加到回退栈。

**Fragment与Activity之间的通信**

Fragment依附于Activity存在，因此与Activity之间的通信可以归纳为以下几种：

* 如果Activity中包含自己管理的Fragment的引用，可以通过访问其public方法进行通信
* 如果Activiy中没有保存任何Fragment的引用，那么没关系，每个Fragment都有一个唯一的TAG或者ID，可以通过getFragmentManager.findFragmentByTag()或者findFragmentById()获得Fragment实例，然后进行操作
* Fragment中可以通过getActivity()得到当前绑定的Activity实例，然后进行操作
* setArguments(Bundle) / getArguments()

**通信优化**

接口实现

**如何处理运行时配置发生变化**

横竖屏切换时导致Fragment多次重建绘制：

不要在构造函数中传递参数，最好通过setArguments()传递。

在onCreate(Bundle savedInstanceState)中判断savedInstanceState为空时在重建，当发生重建时，原本的数据如何保持？类似Activity，Fragment也有onSaveInstanceState方法，在此方法中进行保存数据，然后在onCreate或者onCreateView或者onActivityCreated进行恢复都可以。



### 14. <span id="android_base_14">Json、XML</span>

Json是一种轻量级的数据交换格式，具有良好的可读和便于编写的特性。XML即扩展标记语言，用来标记电子文件使其具有结构性的标记语言，可以用来标记数据。定义数据类型，是一种允许用户对自己的标记语言进行定义的源语言。

JSON在编码和解码上优于XML，并且数据体积更下，解析更快，与JavaScript交互更加方便，但是对数据的描述性较XML差。



### 15. <span id="android_base_15">Assets目录与res目录的区别</span>

	assets目录与res下的raw、drawable目录一样，也可用来存放资源文件，但它们三者区别如下：

|             | assets   | res/raw   | res/drawable   |
| ----------- | -------- | --------- | -------------- |
| 获取资源方式      | 文件路径+文件名 | R.raw.xxx | R.drawable.xxx |
| 是否被压缩       | false    | false     | true(失真压缩)     |
| 能否获取子目录下的资源 | true     | false     | false          |

res/raw和assets的区别：

	res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即可，assets文件夹下的文件不会被映射到R文件中，访问的时候需要AssetManager类。
	
	res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹。
	
	读取res/raw下的文件资源，通过以下方式获取输入流来进行写操作：

```java
InputStream is = getResources().openRawResource(R.id.filename);
```

	读取assets下的文件资源，通过以下方式获取输入流进行写操作，使用AssetManager。

注意：

1. AssertManager中不能处理单个超过1M的文件，而raw没有这个限制
2. assets文件夹是存放不进行编译加工的原生文件，即该文件夹里面的文件不会像xml、java文件被预编译，可以存放一些图片、html、js等等



### 16. <span id="android_base_16">View视图绘制过程原理</span>

	View视图绘制需要搞清楚两个问题，一个是从哪里开始绘制，一个是怎么绘制？

从哪里开始绘制？我们平常使用Activity的时候，都会调用setContentView来设置布局文件，没错，视图绘制就是从这个方法开始。

怎么绘制？

在我们的Activity中调用了setContentView之后，会转而执行PhoneWindow的setContentView，在这个方法里面会判断我们存放内容的ViewGroup（这个ViewGroup可以是DecorView也可以是DecorView的子View）是否存在。不存在的话，则会创建一个DecorView处理，并且会创建出相应的窗体风格，存在的话则会删除原先的ViewGroup上面已有的View，接着会调用LayoutInflater的inflate方法以pull解析的方式将当前布局文件中存在的View通过addView的方式添加到ViewGroup上面来，接着在addView方法里面就会执行我们常见的invalidate方法了，这个方法不只是在View视图绘制的过程中经常用到，其实动画的实现原理也是不断的调用这个方法来实现视图不断重绘的，执行这个方法的时候会调用父View的invalidateChild方法，这个方法是属于ViewParent的，ViewGroup以及ViewRootImpl中都会他进行了实现，invalidateChild里面主要做的是就是通过do while循环一层一层计算出当前View的四个点所对应的矩阵在ViewRoot中所对应的位置，那么有了这个矩阵的位置之后最终都会执行ViewRootImpl的invalidateChildInParent方法，执行这个方法的时候首先会检查当前线程是不是主线程，因为我们要开始准备更新UI了，不是主线程的话是不允许更新UI的，接着就会执行scheduleTraversals方法了，这个方法会通过handler来执行doTraversal方法，在这个方法里面就见到了我们平常所熟悉的View视图绘制的起点方法performTraversals了。

那么接下来就是真正的视图绘制流程了，大体上讲View的绘制流程经历了Measure测量、Layout布局以及Draw绘制的三个过程，具体来讲是从ViewRootImpl的performTraversals方法开始，首先执行的将是performMeasure方法，这个方法里面会传入两个MeasureSpec类型的参数，它在很大程度上决定了View的尺寸规格，对于DecorView来说宽高的MeasureSpec值的获取与窗口尺寸以及自身的LayoutParams有关，对于普通View来说其宽高的MeasureSpec值获取由父容器以及自身的LayoutParams属性共同决定，在performMeasure里面会执行measure方法，在measure方法里面会执行onMeasure方法，到这里Measure测量过程对View与ViewGroup来说是没有区别的，但是从onMeasure开始两者有差别了，因为View本身已经不存在子View了，所以他onMeasure方法将执行setMeasuredDimension方法，该方法会设置View的测量值，但是对于ViewGroup来说，因为它里面还存在着子View，那么我们就需要继续测量它里面的子View了，调用的方法是measureChild方法，该方法内部又会执行measure方法，而measure方法转而又会执行onMeasure方法，这样不断的递归进行下去，直到整个View树测量结束，这样performMeasure方法执行结束了。接着便是执行performLayout方法了，performMeasure只是测量出了View树中View的大小了，但是还不知道View的位置，所以也就出现了performLayout方法了，performLayout方法首先会执行layout方法，以确定View自身的位置，如果当前View是ViewGroup的话，则会执行onLayout方法。在onLayout方法里面又会递归的执行layout方法，直到当前遍历到的View不再是ViewGroup为止，这样整个layout布局过程就结束了。在View树中View的大小以及位置都确定之后，接下来就是真正的绘制View显示在界面的过程了，该过程首先从performDraw方法开始，performDraw首先会执行draw方法，在draw方法中首先绘制背景，接着调用onDraw方法绘制自己，如果当前View是ViewGroup的话，还要调用dispatchDraw方法绘制当前ViewGroup的子View，而dispatchDraw方法里面实际上是通过drawChild方法间接调用draw方法形成递归绘制整个View树，直到当前View不再是ViewGroup为止，这样整个View的绘制过程就结束了。

总结：

* ViewRootImpl会调用performTraversals()，其内部会调用performMeasure()、performLayout、performDraw
* performMeasure会调用最外层的ViewGroup的measure() --> onMeasure() ，ViewGroup的onMeasure()是抽象方法，但其提供了measureChildren()，这之中会遍历子View然后循环调用measureChild()，传入MeasureSpec参数，然后调用子View的measure()到View的onMeasure() -->setMeasureDimension(getDefaultSize(),getDefaultSize())，getDefaultSize()默然返回measureSpec的测量数值，所以继承View进行自定义的wrap_content需要重写。
* performLayout()会调用最外层的ViewGroup的layout(l,t,r,b)，本View在其中使用setFrame()设置本View的四个顶点位置。在onLayout(抽象方法)中确定子View的位置，如LinearLayout会遍历子View，循环调用setChildFrame() --> 子View.layout()
* performDraw()会调用最外层的ViewGroup的draw()方法，其中会先后调用background.draw()绘制背景，onDraw(绘制自己)，dispatchDraw(绘制子View)、onDrawScrollBars(绘制装饰)
* MeasureSpec由两位SpecMode(UNSPECIFIED、EXACTLY(对于精确值和match_parent)、AL_MOST(对应warp_content))和三十位SpecSize组成一个int，DecorView的MeasureSpec由窗口大小和其LayoutParams决定，其他View有父View的MeasureSpec和本View的LayoutParams决定。ViewGroup中有getChildMeasureSpec()来获取子View的MeasureSpec。

### 17. <span id="android_base_17">解决滑动冲突的方式？</span>

	在自定义View的过程中经常会遇到滑动冲突问题，一般滑动冲突的类型有三种：（1）外部View滑动方向和内部View滑动方向不一致；（2）外部View滑动方向和内部View滑动方向一致；（3）上述两种情况的嵌套
	
	一般解决滑动冲突都是利用事件分发机制，有两种方式即外部拦截法和内部拦截法：

**外部拦截法：**

实现思路是事件首先是通过父容器的拦截处理，如果父容器不需要该事件，则不拦截，将事件传递到子View上面，如果父容器决定拦截的话，则在父容器的onTouchEvent里面直接处理该事件，这种方法符合事件分发机制；具体实现是修改父容器的onInterceptTouchEvent方法，在达到某一条件的时候，让该方法直接返回true，就可以把事件拦截下来进而调用自己的onTouchEvent方法来处理，但是有一点需要注意的是，如果想让子View能够收到事件，我们需要在onInterceptTouchEvent方法里面判断是DOWN事件的话，就返回false，这样后续的MOVE以及UP事件才有机会传递到子View上面，如果你直接在onInterceptTouchEvent方法里面DOWN情况下返回了true，那么后续的MOVE以及UP事件酱由当前View的onTouchEvent处理了，这样的拦截根本没有意义的，拦截只是在满足一定条件下才会拦截，并不是所有情况下都要拦截。

**内部拦截法：**

实现思路是



### 18. <span id="android_base_18">APP Build过程</span>

Android Studio点击build按钮之后，AS就会编译整个项目，并将apk安装到手机上，这个过程就是Android工程编译打包过程。主要的流程是：

编译 --> DEX --> 打包 --> 签名

**APK构建概述**
![](https://upload-images.jianshu.io/upload_images/2707477-39b6ba4609909c3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

主要有两个过程：

* 编译过程

  输入是本工程的文件以及依赖的各种库文件

  输出是dex文件和编译后的资源文件

* 打包过程

  配合Keystore对上述的输出进行签名对齐，生成最终的apk文件

**APK构建步骤详解**
![](https://upload-images.jianshu.io/upload_images/2707477-43f77a6cdf804a59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/536)

主要分为六个步骤：

1. 通过aapt打包res资源文件，生成R.java文件、resource.arsc和res文件（二进制&非二进制如res/raw和pic保持原样）
2. 处理.aidl文件，生成对应的Java接口文件
3. 通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件
4. 通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex
5. 通过apkbuilder工具，将aapt生成的resource.arsc和res文件、assets文件和classes.dex一起打包生成apk
6. 通过Jarsigner工具，对上面的apk进行debug或release签名
7. 通过zipalign工具，将签名后的apk进行对齐处理

参考自：

[https://www.jianshu.com/p/e86aadcb19e0](https://www.jianshu.com/p/e86aadcb19e0)

[http://mouxuejie.com/blog/2016-08-04/build-and-package-flow-introduction/](http://mouxuejie.com/blog/2016-08-04/build-and-package-flow-introduction/)



### 19. <span id="android_base_19">Android利用scheme协议进行跳转</span>

scheme是一种页面跳转协议。

通过定义自己的scheme协议，可以非常方便的跳转App中的各个页面；

通过scheme协议，服务器可以定制化告诉App跳转到App内部页面。

**Scheme协议在Android中的使用场景**

* H5跳转到native页面
* 客户端获取push消息中后，点击消息跳转到App内部页面
* App根据URL跳转到另外一个App指定页面

**示例**
注册 SchemeActivity：

```xml
<activity android:name=".SchemeActivity">
    <intent-filter>
        //配置协议
        <data android:scheme="scheme" android:host="mtime" android:path="/goodsDetail"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.BROWSABLE"/>
    </intent-filter>
</activity>
```

在MainActivity里通过以下代码跳转到SchemeActivty：

```java
String url = "scheme://mtime/goodsDetail?goodsId=2333";
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
startActivity(intent);
```

然后我们可以在SchemeActivity获取到请求的参数等等信息：

```java
Uri uri = getIntent().getData();
assert uri != null;
String host = uri.getHost();
String path = uri.getPath();
String query = uri.getQuery();
String param = uri.getQueryParameter("goodsId");
Toast.makeText(this, host + "  " + path + "  " + query + "  " + param, Toast.LENGTH_SHORT).show();
```

### 20. <span id="android_base_20">MVC、MVP</span>

**MVC模式**

![](http://7xih5c.com1.z0.glb.clouddn.com/15-10-11/13126761.jpg)

**MVP模式**

![](http://7xih5c.com1.z0.glb.clouddn.com/15-10-11/2114527.jpg)

MVP模式核心思想：

>  MVP把Activity中的UI逻辑抽象成View接口，把业务逻辑抽象成Presenter接口，Model类还是原来的Model。

这就是MVP模式，现在Activity的工作简单了，只是用来相应生命周期，其他工作都丢到Presenter中完成，从上图可以看出，Presenter是Model和View之间的桥梁，为了让结构变得更加简单，View并不能直接对Model进行操作，这也是MVP与MVC最大的不同之处。

**MVP模式的作用**

* 分离了视图逻辑和业务逻辑，降低了耦合
* Activity只处理生命周期的任务，代码变得更加简洁
* 视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去，提高了代码的可阅读性
* Presenter被抽象成接口，可以有多种具体的实现，所以方便进行单元测试
* 把业务逻辑抽象到Presenter中去，避免后台线程引用着Activity导致Activity的资源无法被系统回收从而引起内存泄露和OOM

**MVP模式的使用**
![](http://7xih5c.com1.z0.glb.clouddn.com/15-10-12/94032090.jpg)

从上述的UML图可以看出，使用MVP至少经历以下几个步骤：

1. 创建IPresenter接口，把所有业务逻辑的接口都放在这里，并创建它的实现类PresenterCompl（在这里可以方便的查看业务功能，由于接口可以有多种实现所以也方便写单元测试）
2. 创建IView接口，把所有视图逻辑的接口都放在这里，其实现类是当前的Activity/Fragment
3. 由UML图可以看出，Activity里包含了一个IPresenter，而PresenterCompl里又包含了一个IView并且依赖了Model。Activity里只保留了对IPresenter的调用，其他工作全部留到PresenterCompl中实现
4. Model并不是必须有的，但是一定会有View和Presenter


**总结**
MVP模式的整个核心流程：

View与Model并不直接交互，而是使用Presenter作为View与Model之间的桥梁。其实Presenter中同时持有View层的interface的引用以及Model层的引用，而View层持有Presenter层引用。当View层某个界面需要展示某些数据的时候，首先会调用Presenter层的引用，然后Presenter层会调用Model层请求数据，当Model层数据加载成功之后会调用Presenter层的回调方法通知Presenter层数据加载情况，最后Presenter层在调用View层的接口将加载后的数据展示给用户。

![](http://upload-images.jianshu.io/upload_images/3985563-03352e00ce8b4083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 21.<span id="android_base_21"> SurfaceView</span>

SurfaceView继承至View的，与View的主要区别在于：

* View主要适用于主动更新的情况下，而SurfaceView主要适用于被动更新，例如频繁的刷新
* View在主线程中对画面进行更新，而SurfaceView通常会通过一个子线程进行页面的刷新
* View在绘图时没有使用双缓存机制，而SurfaceView在底层机制中已经实现了双缓存机制

**总结一句话就是：如果自定义View需要频繁的刷新，或者刷新时数据处理量比较大，那么就可以考虑使用SurfaceView来取代View。**

```java
/**
 * 用Surface实现画板
 */
public class DrawBoardView extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    private SurfaceHolder mSurfaceHolder;
    private Canvas mCanvas;
    private volatile boolean mIsDrawing;
    private Paint mPaint;
    private Path mPath;

    public DrawBoardView(Context context) {
        super(context);
        init();
    }

    public DrawBoardView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public DrawBoardView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mSurfaceHolder = getHolder();
        mSurfaceHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);

        mPaint = new Paint();
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10);
        mPaint.setAntiAlias(true);
        mPath = new Path();
    }
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        long start = System.currentTimeMillis();
        while (mIsDrawing) {
            draw();
        }
        long end = System.currentTimeMillis();
        // 50 - 100
        if (end - start < 100) {//保证线程运行时间不少于100ms
            try {
                Thread.sleep(100 - (end - start));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    private synchronized void draw() {
        try {
            mCanvas = mSurfaceHolder.lockCanvas();
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath,mPaint);
        } catch (Exception e) {
        } finally {
            if (mCanvas != null){
                mSurfaceHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                mPath.moveTo(event.getX(),event.getY());
                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(event.getX(),event.getY());
                break;
            case MotionEvent.ACTION_UP:

                break;
        }
        return true;
    }

}

```



### 22. <span id="android_base_22">HandlerThread</span>

HandlerThread继承了Thread，所以它本质上是一个Thread，与普通Thread的区别在于，它不仅建立了一个线程，并且创立了消息队列，有自己的looper，可以让我们在自己的线程中分发和处理消息，并对外提供自己这个Looper对象的get方法。

HandlerThread自带的Looper使它可以通过消息队列，来重复使用当前的线程，节省系统资源开销。这是他的优点也是缺点，每一个任务队列都是以队列的方式逐个被执行到，一旦队列中某个任务执行时间过长，那么就会导致后续的任务都会被延时执行。

**使用步骤：**

1. 创建HandlerThread实例并启动该线程

   ```java
   HandlerThread handlerThread = new HandlerThread("myThread");
   handlerThread.start();
   ```

2. 构造循环消息处理机制

   ```Java
   private Handler.Callback mSubCallback = new Handler.Callback() {
           //该接口的实现就是处理异步耗时任务的，因此该方法执行在子线程中
           @Override
           public boolean handleMessage(Message msg) {
                   //doSomething  处理异步耗时任务的
                   mUIHandler.sendMessage(msg1);   //向UI线程发送消息，用于更新UI等操作
           }
       };
   ```

3. 构造子线程的Handler

   ```java
   //由于这里已经获取了workHandle.getLooper()，因此这个Handler是在HandlerThread线程也就是子线程中
   childHandler = new Handler(handlerThread.getLooper(), mSubCallback);

   button.setOnClickListener(new OnClickListener() {
       @Override
       public void onClick(View v) {
           //发送异步耗时任务到HandlerThread中,也就是上面的mSubCallback中
           childHandler.sendMessage(msg);
       }
   });
   ```

4. 构建UI线程Handler处理消息

   ```java
   private Handler mUIHandler = new Handler(){
           @Override
           public void handleMessage(Message msg) {
               //在UI线程中做的事情，比如设置图片什么的
           }
       };
   ```

**实现原理**

	我们知道，在子线程创建一个Handler，需要手动创建Looper，即先Looper.parpare()，完了在Looper.loop()。HandlerThread的run方法：

```java
@Override
    public void run() {
        mTid = Process.myTid();  //获得当前线程的id
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            //发出通知，当前线程已经创建mLooper对象成功，这里主要是通知getLooper方法中的wait
            notifyAll();
        }
        //设置当前线程的优先级
        Process.setThreadPriority(mPriority);
        //该方法实现体是空的，子类可以实现该方法，作用就是在线程循环之前做一些准备工作，当然子类也可以不实现。
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
```

**特点：**

1. HandlerThread其实是Handler+Thread+Looper的组合，它本质上是一个Thread，因为它继承了Thread，相比普通的Thread，他不会阻塞，因为它内部通过Looper实现了消息循环机制，保证了多个任务的串行执行。缺点：效率低。
2. HandlerThread需要搭配Handler使用，单独使用的意义不大。HandlerThread线程中作具体的事情，必须要在Handler的callback接口中进行，他自己的run方法被写死了。
3. 子线程中的Handler与HandlerThread的联系是通过childHandler=new Handler(handlerThread.getLooper(),mSubCallback)执行的，也就是说，childHandler获得HandlerThread线程的Looper，这样他们两个就在同一个阵营了，这也就是创建Handler作为HandlerThread线程消息执行者，必须在调用start()方法之后的原因 —— HandlerThread.start()之后，run方法才能跑起来，Looper才得以创建，handlerThread.getLooper()才不会出错。
4. HandlerThread会将通过handleMessage传递进来的任务进行串行执行，由messageQueue的特性决定的，从而也说明了HandlerThread效率相对比较低。

### 23. <span id="android_base_23">IntentService</span>

	IntentService是继承并处理异步请求的一个类，其本质上是一个Service，因为它是继承至Service，所以开启IntentService和普通的Service一致。但是他和普通的Service不同之处在于它可以处理异步任务，在任务处理完之后会自动结束。另外，我们可以启动多次IntentService，而每一个耗时任务会以工作队列的方式在IntentService的onHandleIntent回调方法中执行，并且是串行执行。其实IntentService的内部是通过HandleThread和Handle来实现异步操作的。

### 24. <span id="android_base_24">谈谈你对Application类的理解</span>

	首先，Application 在一个 Dalvik 虚拟机里面只会存在一个实例。为什么强调说是一个Dalvik虚拟机，而不是一个APP呢？那是因为，一个App有可能有多个Dalvik虚拟机，也就是传说中的多进程模式。在这种模式下，每一个Dalvik都会存在一个Application实例，它们之间没有关系，在A进程Application里面保存的数据不能在B进程的Application获取，因为它们根本不是一个对象。而且被隔离在两个进程里面，所以这里强调的是一个Dalvik虚拟机，而不是一个App。

Application有两个子类，一个是MultiDexApplication，如果你遇到了65535问题，可以选择继承至它，完成相关的工作。另外一个是在TDD（测试用例驱动）开发模式中使用Moke进行测试的时候用到的，可以来替代Application的Moke类MokeApplication。

在应用启动的时候，会首先调用Application.attach()，当然，这个方法是隐藏的，开发者能接触到的第一个被调用的方法其实是Application.onCreate()，我们通常会在这个方法里面完成各种初始化，比如图片加载库、网络请求库等初始化操作。但是最好别在这个方法里面进行太多的耗时操作，因为这会影响App的启动速度，所以对于不必要的操作可以使用异步操作、懒加载、延时加载等策略来减少对UI线程的影响。

除此之外，由于在Context中可以通过getApplicationContext()获取到Application对象，或者是通过Activity.getApplication()、Service.getApplication()获取到Application，所以可以在Application保存全局的数据，供所有的Activity或者Service使用。

使用上面的三种方法获取到的都是同一个Application对象，getApplicationContext()是在Context的实现类ContextImpl中具体实现的，而getApplication()则是在Activity和Service中单独实现的，所以他们的作用域不同，但是获取到的都是同一个Application对象，因为一个Dalvik虚拟机只有一个Application对象。

但是这里存在一个坑，那就是在低内存情况下，Application有可能被销毁，从而导致保存在Application里面的数据被销毁，最后程序错乱，甚至Crash。

所以当你想在Application保存数据的时候，请做好判null，或者选择其他方式保存你的数据，比如存储在硬盘上等等。

另外，在Application中存在几个有用的方法，比如onLowMemory()和onTrimMemory()（Activity里面也存在这两个方法），在这两个方法里面，我们可以实现自己的内存回收逻辑，比如关闭数据库链接、移除图片内存缓存等操作来降低内存消耗，从而降低被系统回收的风险。

最后，就是要注意Application的生命周期，他和Dalvik虚拟机生命周期一样长，所以在进行单例或者静态变量的初始化操作时，一定要用Application作为Context进行初始化，否则会造成内存泄漏的发生。使用Dialog的时候一般使用Activity的Context。但是也可以使用Application作为上下文，前提是你必须设置Window类型为TYPE_SYSTME_DAILOG，并且申请相关权限。这个时候弹出的Dialog是属于整个Application的，弹出这个Dialog的Activty销毁时也不会回收Dialog，只有在Application销毁时，这个Dialog才会自动消失。



### 25. <span id="android_base_25">Android为什么要设计出Bundle而不是直接使用HashMap来进行数据传递？</span>

- Bundle内部是由ArrayMap实现的，ArrayMap的内部实现是两个数组，一个int数组是存储对象数据对应下标，一个对象数组保存key和value，内部使用二分发对key进行排序，所以在添加、删除、查找数据的时候，都会使用二分发查找，只适用于小数据量操作，如果在数据量比较大的情况下，那么它的性能将退化。而HashMap内部则是数组+链表结构，所以在数据量较小的时候，HashMap的Entry Array比ArrayMap占用更多的内存。因为使用Bundle的场景大多数为小数据量，所以相比之下，在这种情况下使用ArrayMap保存数据，在操作速度和内存占用上都具有优势。
- 另外一个原因，则是在Android中如果使用Intent来携带数据的话，需要数据是基本类型或者是可序列化类型，HashMap使用Serializable进行序列化，而Bundle则是使用Pracelable进行序列化。而在Android平台，更推荐使用Pracelable实现序列化，虽然写法复杂，但是开销更小，所以为了更加快速的进行数据的序列化和反序列化，系统封装了Bundle类，方便我们进行数据的传输。



### 26. <span id="android_base_26">SharedPreference在使用过程中有什么注意点？</span>

**commit()和apply()的区别**

- 返回值

  apply是没有返回值的，而commit返回boolean表明修改是否提交成功。

- 操作效率

  apply是将修改数据提交到内存，而后异步真正提交到硬盘磁盘，而commit是同步的提交到硬盘磁盘上。

  因此在多并发commit的时候，会等待正在处理的commit保存到磁盘后再操作，从而降低了效率。而apply只是原子的提交到内存，后面有调用apply的函数的将会直接覆盖前面的内存数据，从一定程度上提高了效

- 建议

  如果对提交的结果不关心的话，建议使用apply，如果需要确保提交成功且有后续操作的话，还是使用commit。

**多进程表现**

	在多进程中，如果要交换数据，不用使用SharedPreference，因为不同版本表现不稳定，推荐使用ContentProvider替代。

在有的文章中，有提到在多进程中使用SharePrefenerce添加标志位（MODE_MULTI_PROCESS）就可以了。这个标志位是2.3（API 9）之前默认支持的，但是在2.3之后，需要多进程的访问的情景，就需要显示的声明出来。

现在这个标志位被废弃了，因为在某些版本上表现不稳定。



### 27. <span id="android_base_27">SQLite有哪些可以优化的地方？</span>

- 创建索引

  索引有助于加快SELECT查询和WHERE子句，但是会减慢使用UPDATE和INSERT语句时的数据输入。索引可以创建或删除，但不会影响数据。

  优点是加快了数据库检索的速度，包括对单表查询、分组查询、排序查询。

  缺点是索引的创建和维护存在消耗，索引会占用物理内存，且随着数据量的增加而增加。在对数据库进行增删改时需要维护索引，所以会对增删改的性能存在影响。

  使用场景：当某字段数据更新频率较低，查询频率较高，经常有范围查询（> < = >= <=）或order by、group by 发生时建议使用索引。并且选择度越大，建立索引越有优势，这里选择度指一个字段中唯一值的数量/总的数量。还有就是经常同时存取多列，且每列都含有重复值可考虑建立复合索引。

- 使用事务

  原子性操作，要么全部成功，要么全部失败；在执行大量数据的插入操作时，避免频繁操作cursor，可以大幅减少insert操作时间，一般为1-2个数量级。

- 查询必要字段

  查询时只取需要的字段和结果集，更多的结果集会消耗更多的时间以及内存，更多的字段会导致更多的内存消耗。

### 28. <span id="android_base_28">嵌滑滑动机制</span>

	我们知道，Android的时间分发机制中，只要有一个控件消费了事件，其他控件就没办法在接受到这个事件了。因此，当有嵌套滑动场景时，我们需要自己动手解决事件冲突。

嵌套滑动机制的基本原理可以认为是事件共享，即当子控件接受到滑动事件，准备要滑动时，会先通知父控件（startNestedScroll）；然后在滑动之前，会先询问父控件是否要滑动（dispatchNestedPreScroll）；如果父控件响应该事件进行了滑动，那么就会通知子控件它具体消耗了多少滑动距离；然后交由子控件处理剩余的滑动距离；最后子控件滑动结束后，如果滑动距离还有剩余，就会在问一下父控件是否需要在继续滑动剩下的距离（dispatchNestedScroll）

###  29. <span id="android_base_29">RecyclerView 优化</span>

1. 尽量减少Item布局嵌套，比如使用ConstraintLayout

2. 可以的话，写死ItemView的布局宽高，然后设置RecyclerView.setHasFixedSize(true)。当知道Adapter内Item的改变不会影响RecyclerView的宽高时，这样做可以避免重新计算大小。

   但是，如果调用notifyDataSetChanged()，大小还是会重新计算（requestLayout）。当调用Adapter的增删改查方法，最后就会根据mHasFixedSize这个值来判断需不需要requestLayout()。

3. 根据需求修改RecyclerView默认的绘制缓存选项

   recyclerView.setItemViewCacheSize(20)

   recyclerView.setDrawingCacheEnabled(true)

   recyclerView.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH)

4. 在onBindViewHolder 或 getView 方法中，减少逻辑判断，减少临时对象创建。例如：复用事件监听，在其方法外部创建监听，可以避免生成大量的临时变量。

5. 避免整个列表的数据更新，只更新受影响的布局。例如，加载更多时，不使用notifyDataSetChanged，而是使用notifyItemRangeInserted(rangeStart,rangeEnd)

