# 介绍
ViewModel的定义：ViewModel旨在以注重生命周期的方式存储和管理界面相关的数据。ViewModel本质上是视图（View）与数据（Model）之间的桥梁。

**ViewModel特点：**
- 生命周期比Activity长，数据可以在屏幕发生旋转、开关键盘等配置更改后继续留存。
- ViewModel中不能持有Activity引用，因此如果需要使用应用的Context（不可以使用Activity的Context，会导致内存泄露）则继承AndroidViewModel

# 实例

如下代码中，ShareViewModel继承ViewModel类，在clickItem()方法中，通过setValue()方法设置LiveData。
```kotlin
//ShareViewModel.kt
class ShareViewModel : ViewModel() {
    val itemLiveData: MutableLiveData<String> by lazy { MutableLiveData<String>() }

    //点击左侧Fragment中的Item发送数据
    fun clickItem(infoStr: String) {
        itemLiveData.value = infoStr
    }
}
```

如下代码中，在Fragment中首先通过ViewModelProvider.get()方法获取ShareViewModel对象，然后注册Observer并监听数据变化。
```kotlin
//右侧详情页Fragment
class DetailFragment : Fragment() {
    lateinit var mTvDetail: TextView

    //Fragment之间通过传入同一个Activity来共享ViewModel
    private val mShareModel by lazy {
        ViewModelProvider(activity!!).get(ShareViewModel::class.java)
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return LayoutInflater.from(context)
            .inflate(R.layout.layout_fragment_detail, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        mTvDetail = view.findViewById(R.id.tv_detail)
        //注册Observer并监听数据变化
        mShareModel.itemLiveData.observe(activity!!, { itemStr ->
            mTvDetail.text = itemStr
        })
    }
}
```

# ViewModel生命周期
![img](https://developer.android.com/static/images/topic/libraries/architecture/viewmodel-lifecycle.png?hl=zh-cn)

我们通常在系统首次调用 activity 对象的 onCreate() 方法时请求 ViewModel。系统可能会在 activity 的整个生命周期内多次调用 onCreate()，如在旋转设备屏幕时。ViewModel 存在的时间范围是从首次请求 ViewModel 直到 activity 完成并销毁。

> ViewModelStore的存取都是间接在ActivityThread中进行并保存在ActivityClientRecord中。在Activity配置变化时，ViewModelStore可以在Activity销毁时得以保存并在重建时重新从lastNonConfigurationInstances中获取，又因为ViewModelStore提供了ViewModel，所以ViewModel也可以在Activity配置变化时得以保存，这也是为什么ViewModel的生命周期比Activity生命周期长的原因了。
