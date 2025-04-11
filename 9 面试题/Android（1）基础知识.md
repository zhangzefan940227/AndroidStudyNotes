
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

#### Service种类划分

按运行地点分类：

| 类别                   | 区别         | 优点                                       | 缺点                                  | 应用                             |
| -------------------- | ---------- | ---------------------------------------- | ----------------------------------- | ------------------------------ |
| 本地服务(Local Service)  | 该服务依附在主进程中 | 服务依附于主进程而不是独立的进程，这样在一定程度上节约了资源，因为Local服务是在同一个进程，所以不需要IPC和AIDL。相应bindService会方便很多。 | 主进程被kill后，服务便会终止                    | 如：音乐播放等不需要常驻的服务                |
| 远程服务(Remote Service) |            | 服务为独立的进程，对应进程名格式为所在包名加上你指定的 android:process 字符串。由于是独立的进程，因此在Activity所在进程被kill的时候，该服务依然在运行，不受其他进程影响，有利于为多个进程提供服务具有较高的灵活性。 | 该服务是独立的进程，会占用一定的资源，并且使用AIDL进行IPC稍麻烦 | 一些提供系统服务的Service，这种Service是常住的 |

安运行类型分类：

| 类别   | 区别                                       | 应用                                       |
| ---- | ---------------------------------------- | ---------------------------------------- |
| 前台服务 | 会在通知栏显示onGoing的Notification              | 当服务被终止的时候，通知栏的Notification也会消失，这样对于用户有一定的通知作用。常见的如音乐播放服务 |
| 后台服务 | 默认的服务即为后台服务，即不会在通知一栏显示onGoing的Notification | 当服务被终止的时候，用户是看不到效果的。某些不需要运行或者终止提供的服务，如天气更新、日期同步等等 |

按使用方式分类：

| 类别                                | 区别                                       |
| --------------------------------- | ---------------------------------------- |
| startService启动的服务                 | 主要用于启动一个服务执行后台任务，不进行通信，停止服务使用stopService |
| bindService启动的服务                  | 启动的服务要进行通信，停止服务使用unbindService           |
| 同时使用startService、bindService启动的服务 | 停止服务应同时使用stopService与unbindService       |

#### Service生命周期

startService() --> onCreate() --> onStartCommand() --> Service running --> onDestory() 

bindService() --> onCreate() --> onBind() --> Service running --> onUnbind() --> onDestory()

**onCreate()：**

系统在Service第一次创建时执行此方法，来执行**只运行一次**的初始化工作，如果service已经运行，这个方法不会调用。

**onStartCommand()：**

每次客户端调用startService()方法启动该Service都会回调该方法(**多次调用**)，一旦这个方法执行，service就启动并且在后台长期运行，通过调用stopSelf()或stopService()来停止服务。

**onBind()：**

当组件调用bindService()想要绑定到service时，系统调用此方法(**一次调用**)，一旦绑定后，下次在调用bindService()不会回调该方法。在你的实现中，你必须提供一个返回一个IBinder来使客户端能够使用它与service通讯，你必须总是实现这个方法，但是如果你不允许绑定，那么你应返回null

**onUnbind()：**

当前组件调用unbindService()，想要解除与service的绑定时系统调用此方法(**一次调用**，一旦解除绑定后，下次再调用unbindService()会抛异常)

**onDestory()：**

系统在service不在被使用并且要销毁的时候调用此方法（**一次调用**）。service应在此方法中释放资源，比如线程，已注册的监听器、接收器等等。

#### 三种情况下Service的生命周期

1. startService / stopService

   生命周期：onCreate --> onStartCommand --> onDestory

   如果一个Service被某个Activity调用Context.startService 方法启动，那么不管是否有Activity使用bindService绑定或unbindService解除绑定到该Service，该Service都在后台运行，直到被调用stopService，或自身的stopSelf方法。当然如果系统资源不足，Android系统也可能结束服务，还有一种方法可以关闭服务，在设置中，通过应用 --> 找到自己应用 --> 停止。

   注意：

   第一次startService会触发onCreate和onStartCommand，以后在服务运行过程中，每次startService都只会触发onStartCommand

   不论startService多少次，stopService一次就会停止服务

2. bindService / unbindService

   生命周期：onCreate --> onBind --> onUnbind --> onDestory

   如果一个Service在某个Activity中被调用bindService方法启动，不论bindService被调用几次，Service的onCreate方法只会执行一次，同时onStartCommand方法始终不会调用。

   当建立连接后，Service会一直运行，除非调用unbindService来解除绑定、断开连接或调用该Service的Context不存在了（如Activity被finish --- **即通过bindService启动的Service的生命周期依附于启动它的Context**），系统会在这时候自动停止该Service。

   注意：

   第一次bindService会触发onCreate和inBind，以后在服务运行过程中，每次bindService都不会触发任何回调

3. 混合型

   当一个Service再被启动(startService)的同时又被绑定(bindService)，该Service将会一直在后台运行，不管调用几次，onCreate方法始终只会调用一次，onStartCommand的调用次数与startService调用的次数一致（使用bindService方法不会调用onStartCommand）。同时，调用unBindService将不会停止Service，必须调用stopService或Service自身的stopSelf来停止服务。


#### 三种情况下的应用场景

如果你只是想启动一个后台服务长期进行某项任务，那么使用startService便可以了。

如果你想与正在运行的Service取的联系，那么有两种方法，一种是使用broadcast，另外是使用bindService。前者的缺点是如果交流较为频繁，容易造成性能上的问题，并且BroadcastReceiver本身执行代码的时间是很短的（也许执行到一半，后面的代码便不会执行），而后者则没有这些问题，因此我们肯定选择使用bindService（这个时候便同时使用了startService和bindService了，这在Activity中更新Service的某些运行状态是相当有用的）

如果你的服务只是公开了一个远程接口，供连接上的客户端（Android的Service是C/S架构）远程调用执行方法。这个时候你可以不让服务一开始就运行，而只用bindService，这样在第一次bindService的时候才会创建服务的实例运行它，这会节约很多系统资源，特别是如果你的服务是Remote Service，那么该效果会越明显。



### BroadcastReceiver

广播接收器

作用：用于监听 / 接收 应用发出的广播消息，并作出响应

应用场景：

1. 不同组件之间的通信（包括应用内 / 不同应用之间）
2. 与Android系统在特定情况下的通信，如当电话呼入时，网络可用时
3. 多线程通信

#### 实现原理

* 使用了观察者模式：基于消息的发布/订阅事件模型。

* 模型中有三个角色：消息订阅者（广播接收者）、消息发布者（广播发布者）和消息中心（AMS，即Activity Manager Service）

  ![](http://upload-images.jianshu.io/upload_images/944365-0896ba8d9155140e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 原理描述

  1. 广播接收者通过Binder机制在AMS注册
  2. 广播发送者通过Binder机制向AMS发送广播
  3. AMS根据广播发送者要求，在已注册列表中，寻找合适的广播接收者，寻找依据：IntentFilter / Permission
  4. AMS将广播发送到合适的广播接收者相应的消息循环队列
  5. 广播接收者通过消息循环拿到此广播，并回调onReceive()

  注意：广播发送者和广播接收者的执行是异步的，发出去的广播不会关心有没有接收者接收，也不确定接收者何时能接受到。

#### 具体使用

![](http://upload-images.jianshu.io/upload_images/944365-7c9ff656ebd1b981.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 自定义广播接收者BroadcastReceiver

* 继承至BroadcastReceiver基类

* 重写onReceiver()方法

  广播接收器收到相应广播后，会自动回调onReceiver()方法；一般情况下，onReceiver方法会涉及与其他组件之间的交互，如发送Notification、启动service等等；默认情况下，广播接收器运行在UI线程，因此onReceiver方法不能执行耗时操作，否则可能ANR

#### 广播接收器注册

注册方式分为两种：静态注册和动态注册。

**静态注册：**

在AndroidManifest.xml里通过标签声明：

```xml
<receiver
  android:enabled=["true" | "false"]
  //此broadcastReceiver能否接收其他App的发出的广播
  //默认值是由receiver中有无intent-filter决定的：如果有intent-filter，默认值为true，否则为false
  android:exported=["true" | "false"]
  android:icon="drawable resource"
  android:label="string resource"
  //继承BroadcastReceiver子类的类名
  android:name=".mBroadcastReceiver"
  //具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
  android:permission="string"
  //BroadcastReceiver运行所处的进程
  //默认为app的进程，可以指定独立的进程
  //注：Android四大基本组件都可以通过此属性指定自己的独立进程
  android:process="string" >

  //用于指定此广播接收器将接收的广播类型
  //本示例中给出的是用于接收网络状态改变时发出的广播
  <intent-filter>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>
```

**动态注册：**

在代码中通过调用Context的registerReceiver()方法进行动态注册BroadcastReceiver：

```java
@Override
protected void onResume() {
    super.onResume();

    //实例化BroadcastReceiver子类 &  IntentFilter
    mBroadcastReceiver mBroadcastReceiver = new mBroadcastReceiver();
    IntentFilter intentFilter = new IntentFilter();

    //设置接收广播的类型
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

    //调用Context的registerReceiver（）方法进行动态注册
    registerReceiver(mBroadcastReceiver, intentFilter);
}


//注册广播后，要在相应位置记得销毁广播
//即在onPause() 中unregisterReceiver(mBroadcastReceiver)
//当此Activity实例化时，会动态将MyBroadcastReceiver注册到系统中
//当此Activity销毁时，动态注册的MyBroadcastReceiver将不再接收到相应的广播。
@Override
protected void onPause() {
    super.onPause();
    //销毁在onResume()方法中的广播
    unregisterReceiver(mBroadcastReceiver);
}
```

注意：

动态广播最好在Activity的onResume()注册，onPause()注销，否则会导致内存泄漏，当然，重复注册和重复注销也不允许。

**两种注册方式的区别：**

| 注册方式        | 特点                                       | 应用场景       |
| ----------- | ---------------------------------------- | ---------- |
| 静态注册（常驻广播）  | 常驻，不受任何组件的生命周期影响（应用程序关闭后，如果有信息广播来，程序依旧会被系统调用）缺点：耗电、占内存 | 需要时刻监听广播   |
| 动态注册（非常驻广播） | 非常驻，灵活，跟随组件的生命周期变化（组件结束 = 广播结束，在组件结束前，必须移除广播接收器） | 需要特定时刻监听广播 |

#### 广播发送者向AMS发送广播

##### 广播的发送

* 广播是用意图（Intent）标识
* 定义广播的本质：定义广播所具备的意图（Intent）
* 广播发送：广播发送者将此广播的意图通过sendBroadcast()方法发送出去

##### 广播的类型

广播的类型主要分为5类：

* 普通广播（Normal Broadcast）
* 系统广播（System Broadcast）
* 有序广播（Ordered Broadcast）
* 粘性广播（Sticky Broadcast）
* App应用内广播（Local Broadcast）

**普通广播：**

即开发者自身定义Intent的广播（最常用），发送广播使用如下：

```java
Intent intent = new Intent();
//对应BroadcastReceiver中intentFilter的action
intent.setAction(BROADCAST_ACTION);
//发送广播
sendBroadcast(intent);
```

* 若被注册了的广播接收者中注册时IntentFilter的action与上述匹配，则会接收此广播（即进行回调onReceiver()），如下mBroadcastReceiver则会接收上述广播：

  ```xml
  <receiver 
      //此广播接收者类是mBroadcastReceiver
      android:name=".mBroadcastReceiver" >
      //用于接收网络状态改变时发出的广播
      <intent-filter>
          <action android:name="BROADCAST_ACTION" />
      </intent-filter>
  </receiver>
  ```

* 若发送广播有相应权限，那么广播接收者也需要相应权限

##### 系统广播

* Android中内置了很多系统广播：只要涉及到手机的基本操作（如开关机、网络状态变化、拍照等等），都会发出相应的广播

* 每个广播都有特定的IntentFilter（包括具体的action）Android系统广播action如下：

  | 系统操作                                     | action                               |
  | ---------------------------------------- | ------------------------------------ |
  | 监听网络变化                                   | android.net.conn.CONNECTIVITY_CHANGE |
  | 关闭或打开飞行模式                                | Intent.ACTION_AIRPLANE_MODE_CHANGED  |
  | 充电时或电量发生变化                               | Intent.ACTION_BATTERY_CHANGED        |
  | 电池电量低                                    | Intent.ACTION_BATTERY_LOW            |
  | 电池电量充足（即从电量低变化到饱满时会发出广播）                 | Intent.ACTION_BATTERY_OKAY           |
  | 系统启动完成后（仅广播一次）                           | Intent.ACTION_BOOT_COMPLETED         |
  | 按下照相时的拍照按键（硬件按键）时                        | Intent.ACTION_CAMERA_BUTTON          |
  | 屏幕锁屏                                     | Intent.ACTION_CLOSE_SYSTEM_DIALOGS   |
  | 设备当前设置被改变时（设备语言、设备方向等）                   | Intent.ACTION_CONFIGURATION_CHANGED  |
  | 插入耳机时                                    | Intent.ACTION_HEADSET_PLUG           |
  | 未正确移除SD卡但已取出来时（正确移除方法：设置SD卡和设备内存--卸载SD卡） | Intent.ACTION_MEDIA_BAD_REMOVAL      |
  | 插入外部存储装置（如SD卡）                           | Intent.ACTION_MEDIA_CHECKING         |
  | 成功安装APK                                  | Intent.ACTION_PACKAGE_ADDED          |
  | 成功删除APK                                  | Intent.ACTION_PACKAGE_REMOVE         |
  | 重启设备                                     | Intent.ACTION_REBOOT                 |
  | 屏幕被关闭                                    | Intent.ACTION_SCREEN_OFF             |
  | 屏幕被打开                                    | Intent.ACTION_SCREENT_ON             |
  | 关闭系统时                                    | Intent.ACTION_SHUTDOWN               |
  | 重启设备                                     | Intent.ACTION_REBOOT                 |

  注：当使用系统广播时，只需要在注册广播接收者时定义相关的action即可，并不需要手动发送广播，当系统有相关操作时会自动进行系统广播

##### 有序广播

* 定义：发送出去的广播被接收者按照先后顺序接收，有序是针对广播接收者而言的
* 广播接收者接收广播的顺序规则（同时面向静态和动态注册的广播接收者）
  * 按照Priority属性值从大到小排序
  * Priority属性相同者，动态注册的广播优先
* 特点：
  * 接收广播按顺序接收
  * 先接收的广播接收者可以对广播进行截断，即后接收的广播接收者不在接收到此广播
  * 先接收的广播接收者可以对广播进行修改，那么后接收的广播接收者将接收到被修改后的广播
* 具体使用有序广播的使用过程和普通广播非常类似，差异仅在于广播的发送方式：sendOrderedBroadcast(intent);

##### APP应用内广播

* Android中的广播可以跨App直接通信(exported对于有intent-filter情况下默认为true)

* 冲突可能出现的问题：
  * 其他App针对性发出与当前App intent-filter 相匹配的广播，由此导致当前App不断接收广播并处理；
  * 其他App注册与当前App一致的intent-filter用于接收广播，获取广播具体信息。即会出现安全性&效率性的问题

* 解决方案 使用App应用内广播（Local Broadcast）
  * App应用内广播可以理解为一种局部广播，广播的发送者和接收者都同属于一个App
  * 相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 & 效率高

* 具体使用1 将全局广播设置成局部广播
  * 注册广播时将exported属性设置为false，使得非本App内部发出的此广播不被接受
  * 在广播发送和接收时，增设相应权限permission，用于权限验证
  * 发送广播时指定该广播接收器所在的包名，此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中，通过intent.setPackage(packageName)指定包名

* 具体使用2 使用封装好的LocalBroadcastManager类

  使用方式上与全局广播几乎相同，只是注册 / 取消注册广播接收器和发送广播时将参数的context变成了LocalBroadcastManager的单一实例

  注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册，不能静态注册。

  ```java
  //注册应用内广播接收器
  //步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
  mBroadcastReceiver = new mBroadcastReceiver(); 
  IntentFilter intentFilter = new IntentFilter(); 
  
  //步骤2：实例化LocalBroadcastManager的实例
  localBroadcastManager = LocalBroadcastManager.getInstance(this);
  
  //步骤3：设置接收广播的类型 
  intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
  
  //步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
  localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
  
  //取消注册应用内广播接收器
  localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
  
  //发送应用内广播
  Intent intent = new Intent();
  intent.setAction(BROADCAST_ACTION);
  localBroadcastManager.sendBroadcast(intent);
  ```

##### 粘性广播

Android 5.0 & API 21中已经失效



### ContentProvider

内容提供者

#### 作用

进程间进行数据交互&共享，即跨进程通信

![](http://upload-images.jianshu.io/upload_images/944365-3c4339c5f1d4a0fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 原理

ContentProvider的底层采用Android中的Binder机制

#### 具体使用

![](http://upload-images.jianshu.io/upload_images/944365-5c9b0e2ebed36c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 统一资源标识符（RUI）

作用：唯一标识ContentProvider & 其中的数据，外界进程通过URI找到对应的ContentProvider & 其中的数据，再进行数据操作

具体使用：

URI分为系统预置 & 自定义，分别对应系统内置的数据（如通讯录、日程表等等）和自定义数据库。

![](http://upload-images.jianshu.io/upload_images/944365-96019a2054eb27cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```java
// 设置URI
Uri uri = Uri.parse("content://com.carson.provider/User/1") 
// 上述URI指向的资源是：名为 `com.carson.provider`的`ContentProvider` 中表名 为`User` 中的 `id`为1的数据

// 特别注意：URI模式存在匹配通配符* & ＃

// *：匹配任意长度的任何有效字符的字符串
// 以下的URI 表示 匹配provider的任何内容
content://com.example.app.provider/* 
// ＃：匹配任意长度的数字字符的字符串
// 以下的URI 表示 匹配provider中的table表的所有行
content://com.example.app.provider/table/#
```

##### MIME数据类型

* MIME，即多功能Internet邮件扩充服务。它是一种多用途网际邮件扩充协议。MIME类型就是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。

* 作用：指定某个扩展名的文件用某种应用程序来打开。

* ContentProvider根据URI返回MIME类型

  ```
  ContentProvider.getType(uri);
  ```

* MIME类型组成

  每种MIME类型由两部分组成=类型+子类型

  MIME类型是一个包含两部分的字符串，例：

  text / html 类型为text 子类型为html

  text/css    text/xml 等等

  

### 3. <span id="android_base_3">Context的理解？</span>

Android应用模型是基于组件的应用设计模式，组件的运行要有一个完整的Android工程环境。在这个工程环境下，Activity、Service等系统组件才能够正常工作，而这些组件并不能采用普通的Java对象创建方式，new一下就能创建实例了，而是要有它们各自的上下文环境，也就是Context，Context是维持Android程序中各组件能够正常工作的一个核心功能类。

**如何生动形象的理解Context？**

一个Android程序可以理解为一部电影，Activity、Service、BroadcastReceiver和ContentProvider这四大组件就好比戏了的四个主角，它们是剧组（系统）一开始定好的，主角并不是大街上随便拉个人（new 一个对象）都能演的。有了演员当然也得有摄像机拍摄啊，它们必须通过镜头（Context）才能将戏传给观众，这也就正对应说四大组件必须工作在Context环境下。那么Button、TextView等等控件就相当于群演，显然没那么重用，随便一个路人甲都能演（可以new一个对象），但是它们也必须在面对镜头（工作在Context环境下），所以Button mButtom = new Button(context) 是可以的。

**源码中的Context**

```java
public abstract class Context {
}
```

它是一个纯抽象类，那就看看它的实现类。

![](![img](http://upload-images.jianshu.io/upload_images/1187237-1b4c0cd31fd0193f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

它有两个具体实现类：ContextImpl和ContextWrapper。

其中ContextWrapper类，是一个包装类而已，ContextWrapper构造函数中必须包含一个真正的Context引用，同时ContextWrapper中提供了attachBaseContext()用于给ContextWrapper对象指定真正的Context对象，调用ContextWrapper的方法都会被转向其包含的真正的Context对象。ContextThemeWrapper类，其内部包含了与主题Theme相关的接口，这里所说的主题就是指在AndroidManifest,xml中通过android:theme为Application元素或者Activity元素指定的主题。当然，只有Activity才需要主题，Service是不需要主题的，所以Service直接继承与ContextWrapper，Application同理。而ContextImpl类则真正实现了Context中的所有函数，应用程序中所调用的各种Context类的方法，其实现均来源于该类。**Context得两个子类分工明确，其中ContextImpl是Context的具体实现类，ContextWrapper是Context的包装类。** Activity、Application、Service虽都继承自ContextWrapper（Activity继承自ContextWrapper的子类ContextThemeWrapper），但它们初始化的过程中都会创建ContextImpl对象，由ContextImpl实现Context中的方法。

**一个应用程序有几个Context？**

在应用程序中Context的具体实现子类就是：Activity、Service和Application。那么Context数量=Activity数量+Service数量+1。那么为什么四大组件中只有Activity和Service持有Context呢？BroadcastReceiver和ContextPrivider并不是Context的子类，它们所持有的Context都是其他地方传过去的，所以并不计入Context总数。

**Context能干什么？**

Context能实现的功能太多了，弹出Toast、启动Activity、启动Service、发送广播、启动数据库等等都要用到Context。

```java
TextView tv = new TextView(getContext());

ListAdapter adapter = new SimpleCursorAdapter(getApplicationContext(), ...);

AudioManager am = (AudioManager) getContext().getSystemService(Context.AUDIO_SERVICE);getApplicationContext().getSharedPreferences(name, mode);

getApplicationContext().getContentResolver().query(uri, ...);

getContext().getResources().getDisplayMetrics().widthPixels * 5 / 8;

getContext().startActivity(intent);

getContext().startService(intent);

getContext().sendBroadcast(intent);
```

**Context的作用域**

虽然Context神通广大，但并不是随便拿到一个Context实例就可以为所欲为，它的使用还是有一些规则限制的。由于Context的具体实例是由ContextImpl类去实现的，因此在绝大多数场景下，Activity、Service和Application这三种类型的Context都是可以通用的。不过有几种场景比较特殊，比如启动Activity，还有弹出Dialog。出于安全原因的考虑，Android是不允许Activity或Dialog凭空出现的，一个Activity的启动必须要建立在另一个Activity的基础之上，也就是以此形成返回栈。而Dialog则必须在一个Activity上面弹出（除非是System Alert类型的Dialog），因此在这种场景下，我们只能使用Activity类型的Context，否则将会报错。

![](http://upload-images.jianshu.io/upload_images/1187237-fb32b0f992da4781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图我们可以发现Activity所持有的Context的作用域最广，无所不能，因此Activity继承至ContextThemeWrapper，而Application和Service继承至ContextWrapper，很显然ContextThemeWrapper在ContextWrapper的基础上又做了一些操作使得Activity变得更强大。着重讲一下不推荐使用的两种情况：

1. 如果我们用ApplicationContext去启动一个LaunchMode为standard的Activity的时候会报错：

   android.util.AndroidRuntimeException: Calling startActivity from outside of an Activity context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?

   这是因为非Activity类型的Context并没有所谓的任务栈，所以待启动的Activity就找不到栈了。解决这个问题的方法就是为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就为它创建一个新的任务栈，而此时Activity是以singleTask模式启动的。所有这种用Application启动Activity的方式都不推荐，Service同Application。

2. 在Application和Service中去LayoutInflate也是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用，这种方式也不推荐使用。

一句话总结：**凡是跟UI相关的，都应该使用Activity作为Context来处理；其他的一些操作，Service、Activity、Application等实例都可以，当然了注意Context引用的持有，防止内存泄露。**

**如何获取Context？**

有四种方法：

1. View.getContext 返回当前View对象的Context对象，通常是当前正在展示的Activity对象。
2. Activity.getApplicationContext 获取当前Activity所在的进程的Context对象，通常我们使用Context对象时，要优先考虑这个全局的进程Context。
3. ContextWrapper.getBaseContext() 用来获取一个ContextWrapper进行装饰之前的Context，可以使用这个方法，这个方法在实际开发中使用的不多，也不建议使用。
4. Activity.this 返回当前Activity实例，如果是UI控件需要使用Activity作为Context对象，但是默认的Toast实际上使用ApplicationContext也可以。

**getApplication()和getApplicationContext()的区别？**

其内存地址是一样的。Application本身就是一个Context，这里获取getApplicationContext得到的结果就是Application本身的实例。getApplication方法的语义性很强，就是用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那么也许在绝大多数情况下我们都是在Activity或者Service中使用Application，但是如果在一些其他的场景，比如BroadcastReceiver中也想获取Application实例，这时就可以借助getApplicationContext方法了。

```java
public class MyReceiver extends BroadcastReceiver{
  @Override
  public void onReceive(Contextcontext,Intentintent){
    Application myApp= (Application)context.getApplicationContext();
  }
}
```



### 4. <span id="android_base_4">AsyncTask详解</span>

#### Android中的线程

在操作系统中，线程是操作系统调度的最小单元，同时线程又是一种受限的系统资源，即线程不可能无限制的产生，并且**线程的创建和销毁都会有相应的开销**。当系统中存在大量的线程时，系统会通过时间片轮转的方式调度每个线程，因此线程不可能做到绝对的并行。

如果在一个进程中频繁的创建和销毁线程，显然不是高效地做法。正确的做法是采用线程池，一个线程池会缓存一定数量的线程，通过线程池就可以避免因为频繁创建和销毁线程所带来的系统开销。

#### AsyncTask简介

AsyncTask是一个抽象类，它是由Android封装的一个轻量级异步类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。

AsyncTask的内部封装了两个线程池（SerialExecutor和THREAD_POOL_EXECUTOR）和一个Handle（InternalHandler）。

其中SerialExecutor线程池用于任务的排队，让需要执行的多个耗时任务，按顺序排列，THREAD_POLL_EXECUTOR线程池才真正的执行任务，InternalHandler用于从工作线程切换到主线程。

##### AsyncTask的泛型参数

AsyncTask的类声明如下：

```java
public abstract class AsyncTask<Params,Progress,Result>
```

AsyncTask是一个抽象泛型类。

Params：开始异步任务时传入的参数类型

Progress：异步任务执行过程中，返回下载进度值的类型

Result：异步任务执行完成后，返回的结果类型

如果AsyncTask确定不需要传递具体参数，那么这三个泛型参数可以用Void来代替。

##### AsyncTask的核心方法

**onPreExecute()**

这个方法会在**后台任务开始执行之前调用，在主线程执行**。用于进行一些界面上的初始化操作，比如显示一个进度条对话框等等。

**doInBackground(Params...)**

这个方法中的所有代码都会在子线程中运行，我们应该在这里去处理所有的耗时任务。

任务一旦完成就可以通过return语句来将任务的执行结果进行返回，如果AsyncTask的第三个泛型参数指定的是Void，就可以不返回任务执行结果。**注意，这个方法中是不可以进行UI操作的，如果需要更新UI元素，比如说反馈当前任务的执行进度，可以调用publishProgress(Progress ...)方法来完成。**

**onProgressUpdate(Progress...)**

当在后台任务中调用了publishProgress(Progress...)方法后，这个方法就很快被调用，方法中携带的参数就是在后台任务中传递过来的。**在这个方法中可以对UI进行操作，在主线程中进行，利用参数中的数值就可以对界面元素进行相应的更新。**

**onPostExecute(Result)**

当doInBackground(Params...)执行完毕并通过return语句进行返回时，这个方法就很快被调用。返回的数据会作为参数传递到此方法中，**可以利用返回的数据来进行一些UI操作，在主线程中进行，比如说提醒任务执行的结果，以及关闭掉进度条对话框等等。**

上面几个方法的调用顺序为：onPreExecute() --> doInBackground() --> publishProgress() --> onProgressUpdate() --> onPostExecute()

如果不需要执行更新进度则为：onPreExecute() --> doInBackground() --> onPostExecute()

除了上面四个方法，AsyncTask还提供了onCancelled()方法，**它同样在主线程中执行，当异步任务取消时，onCancelled会被调用，这个时候onPostExecute()则不会调用，但是，AsyncTask的cancel()方法并不是真正的去取消任务，只是设置这个任务为取消状态，我们需要在doInBackground()判断终止任务。就好比想要终止一个线程，调用interrupt()方法，只是进行标记为中断，需要在线程内部进行标记判断然后中断线程。**

##### 使用AsyncTask的注意事项

1. 异步任务的实例必须在UI线程中创建，即AsyncTask对象必须在UI线程中创建。
2. execute(Params ... params)方法必须在UI线程中调用。
3. 不要手动调用onPreExecute()、doInBackground(Params ... params)、onProgressUpdate()、onPostExecute() 这几个方法。
4. 不能在doInBackground()中更改UI组件信息。
5. 一个任务实例只能执行一次，如果执行第二次将会抛异常。



### 5. <span id="android_base_5">Android虚拟机以及编译过程</span>

#### 什么是Dalvik虚拟机？

Dalvik是Google公司自己设计用于Android平台的Java虚拟机，它是Android平台的重要组成部分，支持dex格式的Java应用程序的运行。dex格式是专门为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统。Google对其进行了特定的优化，是的Dalvik具有**高效、简洁、节省资源**的特点。从Android系统架构图知，Dalvik虚拟机运行在Android的运行时库层。

Dalvik作为面向Linux、为嵌入式操作系统设计的虚拟机，主要负责完成对象生命周期、堆栈管理、线程管理、安全和异常管理，以及垃圾回收等。另外，Dalvik早期并没有JIT编译器，知道Android2.2才加入了对JIT的技术支持。

#### Dalvik虚拟机的特点

体积小，占用内存空间小。

专有的DEX可执行文件格式，体积更小，执行速度更快。

常量池采用32位索引值，寻址类方法名、字段名，常量更快。。

基于寄存器架构，并拥有一套完整的指令系统。

提供了对象生命周期管理，堆栈管理，线程管理，安全和异常管理以及垃圾回收等重要功能。

所有的Android程序都运行在Android系统进程里，每个进程对应着一个Dalvik虚拟机实例。

#### Dalvik虚拟机与Java虚拟机的区别

Dalvik虚拟机与传统的Java虚拟机有着许多不同点，两者并不兼容，它们显著的不同点主要表现在以下几个方面：

**Java虚拟机运行的是Java字节码，Dalvik虚拟机运行的是Dalvik字节码。**

传统的Java程序经过编译，生成Java字节码保存在class文件中，Java虚拟机通过解码class文件中的内容来运行程序。而Dalvik虚拟机运行的是Dalvik字节码，所有的Dalvik字节码由Java字节码转换而来，并被打包到一个DEX可执行文件中。Dalvik虚拟机通过解码DEX文件来执行这些字节码。

![](http://upload-images.jianshu.io/upload_images/3985563-9deada32508b8ee5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Dalvik可执行文件体积小，Android SDK中有一个叫dx的工具负责将Java字节码转换为Dalvik字节码。**

消除其中的冗余信息，重新组合形成一个常量池，所有的类文件共享同一个常量池。由于dx工具对常量池的压缩，使得相同的字符串常量在DEX文件中只出现一次，从而减小了文件的体积。

简单来讲，dex格式文件就是将多个class文件中公有的部分统一存放，去除冗余信息。

**Java虚拟机与Dalvik虚拟机架构不同，这也是Dalvik与JVM之间最大的区别。**

**Java虚拟机基于栈架构。**程序在运行时虚拟机需要频繁的从栈上读取或写入数据，这个过程需要更多的指令分配与内存访问次数，会耗费不少CPU时间，对于像手机设备资源有限来说，这是相当大的一笔开销。

**Dalvik虚拟机基于寄存器架构。**

数据的访问通过寄存器间直接传递，这样的访问方式比基于栈方式要快很多。

#### Dalvik虚拟机的结构

![](http://upload-images.jianshu.io/upload_images/3985563-4da3de576e6a045d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个应用首先经过DX工具将class文件转换成Dalvik虚拟机可以执行的dex文件，然后由类加载器加载原生类和Java类，接着由解释器根据指令集对Dalvik字节码进行解释、执行。最后，根据dvm_arch参数选择编译的目标机体系结构。

#### Android APK编译打包流程

![](http://upload-images.jianshu.io/upload_images/3985563-cdba319dab32d0c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. AAPT（Android Asset Packaging Tools）工具会打包应用中的资源文件，如AndroidManifest.xml、layout布局中的xml等，并将xml文件编译成二进制形式，当然assets文件夹中的文件不会被编译，图片以及raw文件夹中的资源也会保持原有的形态，需要注意的是raw文件夹中的资源也会生成资源ID。AAPT编译完成后会生成R.java文件。
2. AIDL工会将所有的aidl接口转换为java接口。
3. 所有的Java源代码、R文件、接口都会编译器编译成.class文件。
4. Dex工具会将上述产生的.class文件以及第三方库和其他class文件转化为dex（Dalvik虚拟机可执行文件）文件，dex文件最终会被打包进APK文件。
5. apkbuilder会把编译后的资源和其他资源文件同dex文件一起打入APK中。
6. 生成APK文件之后，，需要对其签名才能安装到设备上，平时测试都会使用debug keystore，当发布应用时必须使用release版的keystore对应用进行签名。
7. 如果对APK正式签名，还需要使用zipalign工具对APK进行对齐操作，这样做的好处是当应用运行时能提高速度，但是会相应的增加内存开销。

**总结：编译 --> DEX --> 打包 --> 签名和对齐**


#### ART虚拟机与Dalvik虚拟机的区别

##### 什么是ART？

ART代表Android Runtime，其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time（JIT）编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。ART则完全改变了这套做法，在应用安装时就预编译字节码到机器语言，这一机制叫Ahead-Of-Time（AOT）编译。在移除解释代码这一过程后，应用程序执行将更加效率。启动更快。

##### ART优点：

1. 系统性能的显著提升。
2. 应用启动更快、运行更快、体验更流畅、触摸反馈更及时。
3. 更长的电池续航能力
4. 支持更低的硬件。

##### ART缺点：

1. 更大的存储空间占用，可能会增加10%-20%
2. 更长的应用安装时间

#### ART虚拟机相对于Dalvik虚拟机的提升

##### 预编译

在Dalvik中，如同其他大多数JVM一样，都采用的是JIT来做及时翻译（动态翻译），将dex或odex中并排的Dalvik code（或者叫smali指令集）运行态翻译成native code去执行。JIT的引入使得Dalvik提升了3-6倍的性能。

而在ART中，完全抛弃了Dalvik的JIT，使用了AOT直接在安装时将其完全翻译成native code。这一技术的引入，使得虚拟机执行指令的速度又一重大提升。

##### 垃圾回收机制

首先介绍下Dalvik的GC过程，主要有四个过程：

1. 当GC被触发时候，其会去查找所有活动的对象，这个时候整个程序与虚拟机内部的所有线程就会挂起，这样目的是在较少的堆栈里找到所引用的对象。
2. GC对符合条件的对象进行标记。
3. GC对标记的对象进行回收。
4. 恢复所有线程的执行现场继续运行。

**Dalvik这么做的好处是，当pause了之后，GC势必是相当快速的，但是如果出现GC频繁并且内存吃紧势必会导致UI卡顿、掉帧、操作不流畅等等。**

后来ART改善了这种GC方式，主要的改善点在将其**非并发过程改成了部分并发，还有就是对内存的重新分配管理。**

当ART GC发生时：

1. GC将会锁住Java堆，扫描并进行标记。
2. 标记完毕释放掉Java堆的锁，并且挂起所有线程。
3. GC对标记的对象进行回收。
4. 恢复所有线程的执行继续运行。
5. **重复2-4直到结束。**

可以看出整个过程做到了部分并发使得时间缩短，GC效率提高两倍。

##### 提高内存使用，减少碎片化

Dalvik内存管理特点是：内存碎片化严重，当然这也是标记清除算法带来的弊端。

**ART的解决：** 在ART中，它将Java分了一块空间命名为 Large-Object-Space，这个内存空间的引入用来专文存放大对象，同时ART又引入了 moving collector 的技术，即将不连续的物理内存快进行对齐。对齐之后内存碎片化就得到了很好的解决。Large-Object-Space的引入是因为moving collector对大块内存的位移时间成本太高。据官方统计，ART的内存利用率提高了10倍左右，大大提高了内存的利用率。



### 6.<span id="android_base_6"> 进程保活方案</span>

#### 保活的两个方案

* 提高进程优先级，降低进程被杀死的概率
* 在进程被杀死后，进行拉活

##### 进程的优先级，划分五级：

1. 前台进程（Foreground process）
2. 可见进程（Visible process）
3. 服务进程（Service process）
4. 后台进程（Background process）
5. 空进程（Empty process）

**前台进程一般有以下特点：**

* 拥有用户正在交互的Activity（已调用onResume）
* 拥有某个Service，后者绑定到用户正在交互的Activity
* 拥有正在前台运行的Service（服务已调用startForeground）
* 拥有一个正执行生命周期回调的Service（onCreate，onStart或onDestory）
* 拥有其正执行其onReceive()方法的BroadcastReceiver

#### 提高进程优先级

* 利用Activity提升权限

  监控手机锁屏解锁事件，在屏幕锁屏时启动1像素的Activity，在用户解锁时将Activity销毁，注意该Activity需设计成用户无感知。

* Notification提升权限

  Android中Service的优先级为4，通过setForeground接口可以将后台Service设置为前台Service，使进程的优先级由4提升为2，从而是进程的优先级仅仅低于用户当前正在交互的进程，与可见进程优先级一致，使进程被杀死的概率大大降低。

#### 进程死后拉活

* 利用系统广播拉活

  在发生特定系统事件时，系统会发出相应的广播，通过在AndroidManifest中静态注册对应的广播监听器，即可在发生响应事件时拉活。

* 利用第三方应用广播拉活

  该方案总的设计思想与接收系统广播类似，不同的是该方案为接收第三方Top应用广播。通过反编译第三方Top应用，如微信、支付宝等等，找出它们外发的广播，在应用中进行监听，这样当这些应用发出广播时，就会将我们的应用拉活。

* 利用系统Service机制拉活

  将Service设置为START_STICKY，利用系统机制在Service挂掉后自动拉活。

* 利用Native进程拉活

  利用Linux中的fork机制创建Native进程，在Native进程中监控主进程的存活，当主进程挂掉后，在Native进程中立即对主进程进行拉活。

  原理：在Android中所有进程和系统组件的生命周期受ActivityManagerService的统一管理。而且，通过Linux的fork机制创建的进程为纯Linux进程，其生命周期不受Android管理。

* 利用 JobScheduler机制拉活

  在Android5.0以后系统对Native进程等加强了管理，Native拉活方式失效。系统在Android5.0以后版本提供了 JobScheduler接口，系统会定时调用该进程以使应用进行一些逻辑操作。

* 利用账号同步机制拉活

  Android系统的账号同步机制会定期同步账号进行，该方案目的在于利用同步机制进行进程的拉活。
#### 其他有效的拉活方案

相互唤醒。

**参考：**

[进程保活方案1](https://www.jianshu.com/p/845373586ac1)

[Android进程保活招式大全](http://geek.csdn.net/news/detail/95035)



### 7. <span id="android_base_7">Android 消息机制</span>

#### 消息机制简介

在Android中使用消息机制，我们首先想到的就是Handler。没错，Handler是Android消息机制的上层接口。我们通常只会接触到Handler和Message开完成消息机制，其实内部还有两大助手共同完成消息传递。

#### 消息机制模型

消息机制主要包含：MessageQueue、Handler、Looper和Message这四大部分。

* Message

  需要传递的消息，可以传递数据

* MessageQueue

  消息队列，但是它的内部实现并不是用的队列，实际上是通过一个单链表的数据结构来维护消息列表，因为单链表在插入和删除上比较有优势。主要功能是向消息池传递消息（MessageQueue.enqueueMessage）和取走消息池的消息（MessageQueue.next）

* Handle

  消息辅助类，主要功能是向消息池发送各种消息事件（Handler.sendMessage）和处理相应消息事件（Handler.handleMessage）

* Looper

  不断循环执行（Looper.loop），从MessageQueue中读取消息，按分发机制将消息分发给目标处理者。

#### 消息机制的架构

**运行流程：**

在子线程执行完耗时操作，当Handler发送消息时，将会调用MessageQueue.enqueueMessage，向消息队列中添加消息。当通过Looper.loop开启循环后，会不断的从线程池中读取消息，即调用MessageQueue.next，然后调用目标Handler（即发送该消息的Handler）的dispatchMessage方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用handleMessage方法，接收消息，处理消息。

**MessageQueue、Handler和Looper三者之间的关系：**

每个线程中只能存在一个Looper，Looper是保存在ThreadLocal中。主线程已经创建一个Looper，所以在主线程中不需要在创建Looper，但是在其他线程中需要创建Looper。每个线程中可以有多个Handler，即一个Looper可以处理来自多个Handler的消息。Looper中维护一个MessageQueue，来维护消息队列，消息队列中的Message可以来自不同的Handler。

#### 总结

![](http://upload-images.jianshu.io/upload_images/3985563-b3295b67a2b0477f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Android消息机制之ThreadLocal的工作原理：

Looper中还有一个特殊的概念，那就是ThreadLocal，ThreadLocal并不是线程，它的作用是可以在每个线程中存储数据。大家知道，Handle创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handle内部如何获取当前线程的Looper呢？这就要使用ThreadLocal了，ThreadLocal可以在不同的线程之中互不干扰的存储并提供数据，通过ThreadLocal可以轻松的获取每个线程的Looper。当然，需要注意的是，线程默认是没有Looper的，如果需要使用Handler就必须为线程创建Looper。大家经常提到的主线程，也叫UI线程，它就是ActivityThread，ActivityThread被创建时就会初始化Looper，这也是在主线程默认可以使用Handle的原因。

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说无法获取到数据。在日常开发中用到ThreadLocal的地方很少，但是在某些特殊的场景下，通过ThreadLocal可以轻松的实现一些看起来很复杂的功能，这一点在Android源码中也有所体现，比如Looper、ActivityThread以及AMS中都用到了ThreadLocal。具体到ThreadLocal的使用场景，这个不好统一来描述，一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以采用ThreadLocal。比如对于Handle来说，它需要获取当前线程的Looper，很显然Looper的作用域就是线程并且不同线程具有不同的Looper，这个时候通过ThreadLocal就可以轻松的实现Looper在线程中的存取。

ThreadLocal另外一个使用场景是复杂逻辑下的对象传递，比如监听器的传递，有些时候一个线程中的任务过于复杂，这可能表现为函数调用栈比较深以及代码入口的多样性，在这种情况下，我们又需要监听器能够贯穿整个线程的执行过程，这个时候可以怎么做呢？其实就可以采用ThreadLocal，采用ThreadLocal可以让监听器作为线程内的局部对象而存在，在线程内部只要通过get方法就可以获取到监听器。而如果不采用ThreadLocal，那么我们能想到的可能就是一下两种方法：

1. 将监听器通过参数的形式在函数调用栈中进行传递

   在函数调用栈很深的时候，通过函数参数来传递监听器对象几乎是不可接受的

2. 将监听器作为静态变量供线程访问

   这是可以接受的，但是这种状态是不具有可扩充性的，比如如果同时有两个线程在执行，那就需要提供两个静态的监听器对象，如果有十个线程在并发执行呢？提供十个静态的监听器对象？这显然是不可思议的。而采用ThreadLocal每个监听器对象都在自己的线程内部存储，根本就不会有这个问题。

```
ThreadLocal<Boolean> threadLocal = new ThreadLocal<>();
threadLocal.set(true);
        Log.i(TAG, "getMsg: MainThread"+threadLocal.get());
        new Thread("Thread#1"){
            @Override
            public void run() {
                threadLocal.set(false);
                Log.i(TAG, "run: Thread#1" + threadLocal.get());
            }
        }.start();
        new Thread("Thread#2"){
            @Override
            public void run() {
                Log.i(TAG, "run: Thread#2" + threadLocal.get());
            }
        }.start();
        
输出：true、false、null
```

虽然在不同线程中访问的是同一个ThreadLocal对象，但是它们获取到的值却是不一样的。不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后再从数据中根据当前ThreadLocal的索引去查找出对应的value值，很显然，不同线程中的数组是不同的，这就是为什么通过ThreadLocal可以在不同的线程中维护一套数据副本并且彼此互不干扰。

ThreadLocal是一个泛型类，里面有两个重要方法：get()和set()方法。它们所操作的对象都是当前线程的localValues对象的table数组，因此在不同线程中访问同一个ThreadLocal的get和set方法，它们对ThreadLocal所做的读写操作仅限于各自线程的内部，这就是为什么ThreadLocal可以在多个线程中互不干扰的存储和修改数据。

[https://blog.csdn.net/singwhatiwanna/article/details/48350919](https://blog.csdn.net/singwhatiwanna/article/details/48350919)

