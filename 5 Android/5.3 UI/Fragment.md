# 1 对“第五组件”的理解
> Google 官方文档中说明：
>
> `Fragment`<font style="color:rgb(32, 33, 36);">表示应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。Fragment 不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。Fragment 的视图层次结构会成为宿主的视图层次结构的一部分，或附加到宿主的视图层次结构。</font>
>

从中可知几点：

1. `Fragment`有自己的生命周期，但是不能独立存在
2. 必须托管给`Activity`或者另一个`Fragment`，但是通常并不托管给`Fragment`
3. `Fragment`显示在Activity中，可以成为一个Activity的界面的一部分



不同尺寸屏幕上的两种显示方式

 ![1656661415593-5f23cd88-224a-457b-ac06-5205946b5315.png](img/wrxxI-RmS2IimQKX/1656661415593-5f23cd88-224a-457b-ac06-5205946b5315-824763.png)



# 2 创建并使用 Fragment
首先在 build.gradle 文件中添加依赖：

```java
dependencies {
    def fragment_version = "1.5.0"

    // Java language implementation
    implementation "androidx.fragment:fragment:$fragment_version"
    // Kotlin
    implementation "androidx.fragment:fragment-ktx:$fragment_version"
}
```



创建 Fragment 类，创建 Fragment 类对应的布局文件，并在类中构造函数中引用：

```java
class ExampleFragment extends Fragment {
    public ExampleFragment() {
        super(R.layout.example_fragment);
    }
}
```



## 2.1 静态添加
直接将 Fragment 控件添加到 Activity 对应的布局文件中：

```java
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.example.ExampleFragment" />
```

## 2.2 动态添加
在 Activity 对应的布局文件中指定要添加的 Fragment 控件。

```java
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```



在 Activity 文件中添加 Fragment

```java
public class ExampleActivity extends AppCompatActivity {
    public ExampleActivity() {
        super(R.layout.example_activity);
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
            getSupportFragmentManager().beginTransaction()
                .setReorderingAllowed(true)
                .add(R.id.fragment_container_view, ExampleFragment.class, null)
                //.add(R.id.frameLayout, ExampleFragment.class, null)
                .commit();
        }
    }
}
```

实际上，动态添加 Fragment 是通过代码将 Fragment 替换 example_activity.xml 中指定的某个控件，可以是 FragmentContainerView 也可以是 FrameLayout。如注释行所示例子。

在前面的示例中，请注意片段事务仅在 savedInstanceState 为 null 时创建。这是为了确保在首次创建活动时只添加一次片段。

当配置发生变化并重新创建活动时，savedInstanceState 不再为空，并且不需要第二次添加片段，因为片段会自动从 savedInstanceState 恢复。

## 2.3 Activity与Fragment传递数据
如果需要一些初始数据，则可以通过在对 FragmentTransaction.add() 的调用中提供一个 Bundle 将参数传递进去，如下所示：

```java
public class ExampleActivity extends AppCompatActivity {
    public ExampleActivity() {
        super(R.layout.example_activity);
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
            Bundle bundle = new Bundle();
            bundle.putInt("some_int", 0);

            getSupportFragmentManager().beginTransaction()
                .setReorderingAllowed(true)
                .add(R.id.fragment_container_view, ExampleFragment.class, bundle)
                .commit();
        }
    }
}
```

然后可以通过调用 requireArguments() 从 Fragment 中检索参数 Bundle，并且可以使用适当的 Bundle getter 方法来检索每个参数。

```java
class ExampleFragment extends Fragment {
    public ExampleFragment() {
        super(R.layout.example_fragment);
    }

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        int someInt = requireArguments().getInt("some_int");
        ...
    }
}
```

# 3 FragmentManager
> FragmentManager 类负责对应用的 Fragment 执行一些操作，如添加、移除或替换它们，以及将它们添加到返回堆栈。
>
> 如果需要开发应用的导航栏，可以使用Google官方提供的<font style="color:rgb(32, 33, 36);"> </font>[Jetpack Navigation](https://developer.android.google.cn/guide/navigation)<font style="color:rgb(32, 33, 36);"> </font>库
>



## 3.1 在Activity中访问
每个 FragmentActivity 及其子类（如 AppCompatActivity）都可以通过`getSupportFragmentManager()`方法访问 FragmentManager。

需要注意的是，FragmentActivity 是 Activity 的子类。如果我们新建的 ExampleActivity 继承自 Activity 则不可以调用`getSupportFragmentManager()`方法。

![1656666474127-cbc07bdb-68f5-4974-90eb-d788f0f65947.png](img/wrxxI-RmS2IimQKX/1656666474127-cbc07bdb-68f5-4974-90eb-d788f0f65947-830067.png)

## 3。2 在Fragment中访问
Fragment 也可以托管 Fragment。

通过`getChildFragmentManager()`获取 "管理Fragment子级的" FragmentManager 的引用。

通过`getParentFragmentManager()`访问宿主的 FragmentManager

![1656666610988-b39749ab-e35a-4409-acf0-924f4bef9d02.png](img/wrxxI-RmS2IimQKX/1656666610988-b39749ab-e35a-4409-acf0-924f4bef9d02-337335.png)

**示例1**中宿主（蓝色）Fragment 托管2个 Fragment，构成拆分视图屏幕。

**示例2**中宿主（Fragment）Fragment 托管1个 Fragment，构成[滑动视图](https://developer.android.google.cn/guide/navigation/navigation-swipe-view-2#implement_swipe_views)屏幕。

下图为各个 FragmentManager 的映射。

![1656667010545-0019fdb4-28b2-4e90-bab3-6620d43c90d3.png](img/wrxxI-RmS2IimQKX/1656667010545-0019fdb4-28b2-4e90-bab3-6620d43c90d3-715933.png)

## 3.3 回退栈
Activity的切换是通过任务栈实现的。对于Fragment，如果不手动添加回退栈，它是<font style="color:rgb(64, 64, 64);">直接销毁再重建。</font>

<font style="color:rgb(64, 64, 64);">如果将 Fragment 添加到回退栈中，那么它就有了类似 Activity 的任务栈的操作。</font>

<font style="color:rgb(64, 64, 64);">【示例】</font>

1. <font style="color:rgb(64, 64, 64);">Fragment 1 添加按钮能够跳转到 Fragment 2</font>
2. <font style="color:rgb(64, 64, 64);">Fragment 2 添加按钮能够回退到 Fragment 1</font>

【代码】

1> Activity 中，初始化 Fragment 1，不添加回退栈

```java
Fragment1 f1 = new Fragment1();
FragmentTransaction ft = getFragmentManager()
        .beginTransaction();
ft.add(R.id.fl, f1);
ft.commit();
```

2> Fragment 1 中添加按钮事件，将当前事务添加到回退栈

```java
Fragment2 f2 = new Fragment2();
FragmentManager fm = getFragmentManager();
FragmentTransaction tx = fm.beginTransaction();
tx.replace(R.id.fl, f2);
tx.addToBackStack(null);
tx.commit();
```

3> Fragment 2 中添加按钮事件，回退到 Fragment 1

```java
FragmentManager fm = getFragmentManager();
fm.popBackStack();
```

****

**个人理解**

如果要从B回退到A，就要在A跳转到B时，将该跳转事务添加到回退栈中。

从B回退到A时，相当于将 "跳转到B的事务" 做出栈操作。

# 4 查找 Fragment
## 4.1 findFragmentById
1. 可以使用`findFragmentById()`获取在布局容器中当前 Fragment 的引用。
2. 如果 Fragment 是静态添加到 Activity 中，则可使用`findFragmentById()`按给定的 ID 查找 Fragment
3. 在 FragmentTransaction 中添加时，可使用`findFragmentById()`按容器 ID 进行查找。

【示例】

```java
FragmentManager fragmentManager = getSupportFragmentManager();
fragmentManager.beginTransaction()
    .replace(R.id.fragment_container, ExampleFragment.class, null)
    .setReorderingAllowed(true)
    .addToBackStack(null)
    .commit();

...

ExampleFragment fragment = (ExampleFragment) fragmentManager.
    findFragmentById(R.id.fragment_container);
```

## 4.2 findFragmentByTag
1. 可以提前为 Fragment 分配一个唯一标识，然后使用 findFragmentByTag() 方法获取该 Fragment 的引用。
2. 可以在布局中定义的 Fragment 上添加属性 `android:tag` 分配标记
3. 可以在`FragmentTransaction`中的 add() replace() 操作器件分配标记

```java
FragmentManager fragmentManager = getSupportFragmentManager();
fragmentManager.beginTransaction()
    .replace(R.id.fragment_container, ExampleFragment.class, null, "tag")
    .setReorderingAllowed(true)
    .addToBackStack(null)
    .commit();

...

ExampleFragment fragment = (ExampleFragment) fragmentManager.
    findFragmentByTag("tag");
```

# 5 生命周期
为了管理生命周期，Fragment 实现了 LifecycleOwner，并且拥有一个 Lifecycle 对象，可通过 getLifecycle() 方法访问该对象。

> Lifecycle 状态均表示在枚举类 Lifecycle.State 中：
>
> + INITIALIZED
> + CREATED
> + STARTED
> + RESUMED
> + DESTROYED
>

当一个 Fragment 被实例化，其处于 INITIALIZED 状态。

将 Fragment 添加到 FragmentManager 后，FM会负责确定该 Fragment 应处于什么状态，然后将其转为该状态。

当 Fragment 被添加到 FragmentManager 并附加到其宿主 Activity 时，将调用 onAttach() 回调。

当 Fragment 已从 FragmentManager 中移除并与其宿主 Activity 分离时，将调用 onDetach() 回调。

![1658220994532-10073a26-7dcb-4fc3-895e-ea028b500cdf.png](img/wrxxI-RmS2IimQKX/1658220994532-10073a26-7dcb-4fc3-895e-ea028b500cdf-405784.png)

当 Fragment 在其生命周期内运行，其状态向上和向下移动。例如: 添加到堆栈顶部的 Fragment 生命周期变化为 `CREATED -> STARTED -> RESUMED`。当一个 Fragment 从返回堆栈中弹出时，生命周期变化为 `RESUMED -> STARTED -> CREATED -> DESTROYED`。

如果 Fragment 已经处于 CREATED 状态，说明它已经被添加到 FragmentManager 中并且调用了`onAttach()`方法。

该状态下会回调`onCreate()`方法。方法会接收一个 Bundle 参数：saveInstanceState，其中包含由`onSaveInstanceState()`方法保存的状态。

**注意：**第一次创建 Fragment，其 savedInstanceState 值为 null，后续即使未重写`onSaveInstanceState()`方法，savedInstanceState 也不再会为 null

![1658223332393-fd460c0d-c799-49f3-a3b2-f078e2a6dc0e.png](img/wrxxI-RmS2IimQKX/1658223332393-fd460c0d-c799-49f3-a3b2-f078e2a6dc0e-461120.png)

如图所示，`onStop()`回调的顺序和使用`onSaveInstanceState()`保存状态的顺序因 API 级别而异。对于 API 28 之前的所有 API 级别，`onSaveInstanceState()`在`onStop()`之前调用。对于 API 级别 28 及更高级别，调用顺序是相反的。

# 6 与 Fragment 通信
> <font style="color:rgb(32, 33, 36);">为了正确响应用户事件，或为了共享状态信息，通常需要在 activity 与其 fragment 之间或者两个或更多 fragment 之间具有通信渠道。</font>
>
> <font style="color:rgb(32, 33, 36);">为了保持 Fragment 的独立，不应让 Fragment 与其他 Fragment 或宿主 Activity </font>**<font style="color:rgb(32, 33, 36);">直接通信。</font>**
>

<font style="color:rgb(32, 33, 36);">Fragment 提供两种通信方式：</font>

+ <font style="color:rgb(32, 33, 36);">ViewModel —— 与自定义 API 共享持久化数据</font>
+ <font style="color:rgb(32, 33, 36);">Fragment Result API —— 可放置在 Bundle 中的一次性结果</font>

## 6.1 使用 ViewModel 共享数据
ViewModel用于在多个 Fragment 之间或 Fragment 与其宿主 Activity 之间共享数据

### 6.1.1 与宿主 Activity 共享数据








> 更新: 2023-12-03 18:26:14  
> 原文: <https://www.yuque.com/zhangxiaofani4cu/xih3ez/vgqa11>