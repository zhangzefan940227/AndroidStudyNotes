# kotlin 委托
在委托模式中，有两个对象参与处理同一个请求，接受请求的对象将请求委托给另一个对象来处理。

Kotlin 通过关键字 by 实现委托。

# 类委托
类的委托即：一个类中定义的方法实际是调用另一个类的对象的方法来实现的。即委托给另一个类来实现逻辑，本类实际不实现具体方法逻辑。

在 BaseProxy 声明中，by 子句表示，将 base 保存在 BaseProxy 的对象实例内部，而且编译器将会生成继承自 Base 接口的所有方法, 并将调用转发给 base。

```kotlin
class ByTest {
   

    // 定义一个接口,和一个方法 show()
    interface Base {
   
        fun show()
    }

    // 定义类实现 Base 接口, 并实现 show 方法
    open class BaseImpl : Base {
   
        override fun show() {
   
            AbLogUtil.e("BaseImpl::show()")
        }
    }

    // 定义代理类实现 Base 接口, 构造函数参数是一个 Base 对象
    // by 后跟 Base 对象, 不需要再实现 show()
    class BaseProxy(base: Base) : Base by base {
   
        fun showOther() {
   
            AbLogUtil.e("BaseImpl::showOther()")
        }

    }

    // main 方法
    fun mainGo() {
   
        val base = BaseImpl()
        BaseProxy(base).show()
        BaseProxy(base).showOther()
    }
}
```


# 属性委托

属性委托指的是一个类的某个属性值不是在类中直接进行定义，而是将其托付给一个代理类，从而实现对该类的属性统一管理。

属性委托语法格式：

```kotlin
val/var <属性名>: <类型> by <表达式>
```

by 关键字之后的表达式就是委托, 属性的 get() 方法(以及set() 方法)将被委托给这个对象的 getValue() 和 setValue() 方法。属性委托不必实现任何接口, 但必须提供 getValue() 函数(对于 var属性,还需要 setValue() 函数)。

示例代码如下：
```kotlin
import kotlin.reflect.KProperty
// 定义包含属性委托的类
class Example {
    var p: String by Delegate()
}

// 委托的类
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, 这里委托了 ${property.name} 属性"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$thisRef 的 ${property.name} 属性赋值为 $value")
    }
}
fun main(args: Array<String>) {
    val e = Example()
    println(e.p)     // 访问该属性，调用 getValue() 函数

    e.p = "Runoob"   // 调用 setValue() 函数
    println(e.p)
}
```

# Kotlin内置委托
Kotlin 的标准库中已经内置了很多工厂方法来实现属性的委托。

## Lazy
惰性初始化是一种常见的模式，直到第一次访问该属性的时候，才根据需要创建对象的一部分，当初始化过程消耗大量资源并且在使用对象时并不总是需要数据时，这个非常有用。

lazy() 是一个函数, 接受一个 Lambda 表达式作为参数, 返回一个 Lazy <T> 实例，返回的实例可以作为实现延迟属性的委托： **第一次调用 get() 会执行已传递给 lazy() 的 lamda 表达式并记录结果， 后续调用 get() 只是返回记录的结果。**

示例代码如下：
```kotlin
val lazyValue: String by lazy {
    println("computed!")     // 第一次调用输出，第二次调用不执行
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)   // 第一次执行，执行两次输出表达式
    println(lazyValue)   // 第二次执行，只输出返回值
}
--------------------------------------------------------
执行输出结果：
computed!
Hello
Hello
```

## Observable
observable 可以用于实现观察者模式。

Delegates.observable() 函数接受两个参数: 第一个是初始化值, 第二个是属性值变化事件的响应器(handler)。

在属性赋值后会执行事件的响应器(handler)，它有三个参数：被赋值的属性、旧值和新值：

示例代码如下：
```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("初始值") {
        prop, old, new ->
        println("旧值：$old -> 新值：$new")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "第一次赋值"
    user.name = "第二次赋值"
}
--------------------------------------------------------
执行输出结果：
旧值：初始值 -> 新值：第一次赋值
旧值：第一次赋值 -> 新值：第二次赋值
```

## Map
- Map实现了符合属性委托规则的getValue方法，因此只读属性可委托于map对象。
- MutableMap实现了符合属性委托规则的setValue和getValue方法，因此读写属性可委托于mutableMap对象。
> 这么做的好处是对象本身不负责存储属性值，交给Map对象管理，程序也不需要将对象直接暴露出来，只需要通过map管理对象属性即可。

这经常出现在像解析 JSON 或者做其他"动态"事情的应用中。 在这种情况下，你可以使用映射实例自身作为委托。

只读属性示例代码：
```kotlin
/**
 * 定义一个类，构造器传入map对象
 */
class MapDelegateClass(map: Map<String, String>) {
    val property1 : String by map//只读属性委托于map对象
    val property2 : String by map//只读属性委托于map对象
}

fun main(args: Array<String>) {
    val map = mapOf(
            "property1" to "p1",
            "property2" to "p2"
    )
    val mapDelegateClass = MapDelegateClass(map)
    println(mapDelegateClass.property1)//①
    println(mapDelegateClass.property2)//②
    println(map["property1"])//通过map索引属性值名来获取属性，等同于①
    println(map["property2"])//通过map索引属性值名来获取属性，等同于②
}
--------------------------------------------------------
运行结果：
p1
p2
p1
p2
```

读写属性示例代码：
```kotlin
/**
 * 定义一个类，构造器传入MutableMap对象
 */
class MutableMapDelegate(mutableMap: MutableMap<String, String>) {
    var property1: String by mutableMap//属性1委托于mutableMap对象
    var property2: String by mutableMap//属性2委托于mutableMap对象
}

fun main(args: Array<String>) {
    val mutableMap = mutableMapOf<String, String>()
    mutableMap["property1"] = "p1"//①
    mutableMap["property2"] = "p2"//②
    val mutableMapDelegate = MutableMapDelegate(mutableMap)
    println(mutableMapDelegate.property1)
    println(mutableMapDelegate.property2)
    mutableMapDelegate.property1 = "p1change"//等同于①
    mutableMapDelegate.property2 = "p2change"//等同于②
    println(mutableMapDelegate.property1)
    println(mutableMapDelegate.property2)
}
--------------------------------------------------------
运行结果：
p1
p2
p1change
p2change
```
