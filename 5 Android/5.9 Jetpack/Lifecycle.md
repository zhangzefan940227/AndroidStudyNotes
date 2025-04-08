# 介绍
Lifecycle可以让某一个类变成Activity、Fragment的生命周期观察者类，监听其生命周期的变化并可以做出响应。Lifecycle使得代码更有条理性、精简、易于维护。

Lifecycle中主要有两个角色：

- **LifecycleOwner**: 生命周期拥有者，如Activity/Fragment等类都实现了该接口并通过getLifecycle()获得Lifecycle，进而可通过addObserver()添加观察者。
- **LifecycleObserver**: 生命周期观察者，实现该接口后就可以添加到Lifecycle中，从而在被观察者类生命周期发生改变时能马上收到通知。

实现LifecycleOwner的生命周期拥有者可与实现LifecycleObserver的观察者完美配合。

**主要作用：**将原本需要在Activity/Fragment生命周期方法中对组件依赖进行的操作移入组件本身当中。

# LifecycleObserver

首先看观察者，MyLifeCycleObserver实现LifecycleObserver接口。

```kotlin
open class MyLifeCycleObserver : LifecycleObserver {

    @OnLifecycleEvent(value = Lifecycle.Event.ON_START)
    fun connect(owner: LifecycleOwner) {
        Log.e(JConsts.LIFE_TAG, "Lifecycle.Event.ON_CREATE:connect")
    }

    @OnLifecycleEvent(value = Lifecycle.Event.ON_STOP)
    fun disConnect() {
        Log.e(JConsts.LIFE_TAG, "Lifecycle.Event.ON_DESTROY:disConnect")
    }
}

```

上述代码中，在connect()、disConnect()方法上使用了@OnLifecycleEvent注解，并传入了一种生命周期事件。

- 类型可以为 `ON_CREATE`、`ON_START`、`ON_RESUME`、`ON_PAUSE`、`ON_STOP`、`ON_DESTROY`，对应Activity的生命周期回调。
- 类型也可以为`ON_ANY`，可以匹配任何生命周期回调。

所以在上述代码中，connect()方法在Activity的onStart()触发时执行，disConnect()方法则在Activity的onStop()触发时执行。

我们接着看Activity类。

```kotlin
class MainActivity : AppCompatActivity() {

      override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onCreate")

        //添加LifecycleObserver观察者
        lifecycle.addObserver(MyLifeCycleObserver())
    }

  ...

}
```

上述代码中，MainActivity继承自AppCompatActivity，已经包含了lifecycle对象并已经帮助我们完成了初始化，可以通过getLifecycle()获取Lifecycle实例。

Lifecycle是抽象类，实例化的是子类LifecycleRegistry

```java
public interface LifecycleOwner {
    @NonNull
    Lifecycle getLifecycle();
}

public class LifecycleRegistry extends Lifecycle {

}

```

# 自定义LifecycleOwner

自定义 CustomLifeCycleOwner 实现 LifecycleOwner 

```kotlin
class CustomLifeCycleOwner : LifecycleOwner {
    private lateinit var registry: LifecycleRegistry

    fun init() {
        registry = LifecycleRegistry(this)
        //通过setCurrentState来完成生命周期的传递
        registry.currentState = Lifecycle.State.CREATED
    }

    fun onStart() {
        registry.currentState = Lifecycle.State.STARTED
    }

    fun onResume() {
        registry.currentState = Lifecycle.State.RESUMED
    }

    fun onPause() {
        registry.currentState = Lifecycle.State.STARTED
    }

    fun onStop() {
        registry.currentState = Lifecycle.State.CREATED
    }

    fun onDestroy() {
        registry.currentState = Lifecycle.State.DESTROYED
    }

    override fun getLifecycle(): Lifecycle {
        //返回LifecycleRegistry实例
        return registry
    }
}
```

上述代码中，关键点在于重写了getLifecycle()方法，并返回LifecycleRegistry实例。

接下来通过LifecycleRegistry#setCurrentState传递生命周期状态。

```kotlin
//注意：这里继承的是Activity，本身并不具备LifecycleOwner能力
class MainActivity : Activity() {
    val owner: CustomLifeCycleOwner = CustomLifeCycleOwner()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onCreate")

        //自定义LifecycleOwner
        owner.init()
        //添加LifecycleObserver
        owner.lifecycle.addObserver(MyLifeCycleObserver())
    }

    override fun onStart() {
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onStart")
        super.onStart()
        owner.onStart()
    }

    override fun onResume() {
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onResume")
        super.onResume()
        owner.onResume()
    }

    override fun onPause() {
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onPause")
        super.onPause()
        owner.onPause()
    }

    override fun onStop() {
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onStop")
        super.onStop()
        owner.onStop()
    }

    override fun onDestroy() {
        Log.e(JConsts.LIFE_TAG, "$ACTIVITY:onDestroy")
        super.onDestroy()
        owner.onDestroy()
    }
}

```

# Service中使用Lifecycle

自定义Service继承LifecycleService

```kotlin
//MyService.kt
class MyService : LifecycleService() {
    override fun onCreate() {
        Log.e(JConsts.SERVICE, "Service:onCreate")
        super.onCreate()
        lifecycle.addObserver(MyLifeCycleObserver())
    }

    override fun onStart(intent: Intent?, startId: Int) {
        Log.e(JConsts.SERVICE, "Service:onStart")
        super.onStart(intent, startId)
    }

    override fun onDestroy() {
        Log.e(JConsts.SERVICE, "Service:onDestroy")
        super.onDestroy()
    }
}

//MainActivity.kt
  /**
   * 启动Service
   */
  private fun startLifecycleService() {
      val intent = Intent(this, MyService::class.java)
      startService(intent)
  }

  /**
   * 关闭Service
   */
  fun closeService(view: View) {
      val intent = Intent(this, MyService::class.java)
      stopService(intent)
  }

```

LifecycleService中实现了LifecycleOwner接口，所以子类中可以直接通过getLifecycle()添加生命周期Observer，在Activity中启动Service。


