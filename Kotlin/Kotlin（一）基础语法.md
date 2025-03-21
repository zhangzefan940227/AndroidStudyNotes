# Kotlin（一）基础语法

# 前言
在线运行Kotlin代码的网站：[https://try.kotlinlang.org](https://try.kotlinlang.org)

# 基础语法
## 1 变量和函数
### 1.1 变量
Kotlin 中定义一个变量，只允许在变量前声明两种关键字：val 和 var

+ val（value的简写）用来声明一个不可变的变量，对应 Java 中的 final 变量
+ var（variable的简写）用来声明一个可变的变量，对应 Java 中的非 final 变量

> Kotlin 中有出色的类型推导机制，可以推导出变量是什么类型。
>

**注意：** Kotlin 中每一行代码的结尾不加分号，Kotlin 的类型推导机制并不总是能正常工作，比如对一个变量延迟赋值，这时就需要显式地声明变量类型

语法如下所示：

```kotlin
var a : Int = 10
//可读可写 变量名 : 类型 = 值

val b : String = "hello"
//不可变 变量名 : 类型 = 值
```

这里的 Int 的首字母是大写的，表示 Kotlin 完全摒弃了 Java 的基本数据类型。

Java 八种类型包括：Int、Long、Short、Float、Double、Boolean、Char、Byte

Kotlin 内置数据类型：

+ String、Char、Boolean、Int、Double
+ List		集合
+ Set		无重复的元素集合
+ Map		键值对集合

### 1.2 函数
语法规则如下：

```kotlin
fun methodName(param1: Int, param2: Int): Int {
    return 0
}
```

解释如下：

+ fun：声明函数的关键字
+ methodName：函数名，小驼峰命名
+ param1,param2：两个 Int 型的参数
+ Int：最后的 Int 表示返回值为 Int 类型

### 1.3 常量
语法规则：

```kotlin
const val PI = 3.14	//定义编译时常量
```

**注意**：修饰符 const 不适用于**局部变量**，只能定义全局常量。**编译时常量**顾名思义该常量必须在编译时进行初始化，而如果在方法内定义，则只能在运行时调用该方法赋值，因此只能在方法外定义。

## 2 程序的逻辑控制
### 2.1 if


if 和 Java 中的没什么区别，但是比 Java 中多了一项功能：可以有返回值，返回值就是 if 语句每一个条件中最后一行代码的返回值。看个例子：



```kotlin
fun largerNumber(num1: Int, num2: Int): Int{
    var value = 0
    if (num1 > num2){
        value = num1
    } else {
        value = num2
    }
}
//可以简写成如下格式：
fun largerNumber(num1: Int, num2: Int): Int{
    var value = if (num1 > num2 ) {
        num1
    } else {
        num2
    }
}
```



**语法糖：**当一个函数只有一行代码时，可以省略函数体部分，直接将这一行代码使用等号串连在函数定义的尾部。那么对上述代码进行进一步的精简：



```kotlin
fun largerNumber(num1: Int, num2: Int) = if (num1 > num2) num1 else num2
```



### 2.2 when


一个查询考试成绩的功能，输入学生姓名，返回学生成绩



```kotlin
fun getScore(name: String) = when (name) {
    "Tom" -> 87
    "Jim" -> 94
    "Jack" -> 84
    else -> 0
}
```



when 的格式如下：



```kotlin
匹配值 -> { 执行逻辑 }
```



when 还允许类型匹配，如下示例：



```kotlin
fun checkNumber(num: Number) {
    when (num) {
        is Int -> println("it is Int")
        is Double -> println("it is Double")
    }
}
```



### 2.3 for 循环


用如下代码来表示一段闭区间 [0,10]



```kotlin
val range = 0..10
```



有了区间后，用 for-in 循环来遍历这个区间：



```kotlin
fun main() {
    for (i in 0..10) {
        println(i)
    }
}
```



左闭右开的区间 [0,10) 用如下方式表示：



```kotlin
val range = 0 until 10
```



如果想要跳过一些元素，可以用 step 关键字



```kotlin
fun main() {
    for (i in 0 until 10 step 2) {
        print(i+" ")
    }
}
//output: 0 2 4 6 8
```



还可以用关键字 downTo 创建一个降序区间



```kotlin
fun main() {
    for (i in 0 until 10 downTo 1) {
        print(i+" ")
    }
}
output: 9 8 7 6 5 4 3 2 1
```



## 3 面向对象


### 3.1 类与对象


声明一个类



```kotlin
class Person {
    //类体
}
```



实例化类：



```kotlin
val p = Person()
```



### 3.2 继承与构造函数


#### 继承


声明一个可继承类



```kotlin
open class Person {
    //TODO
}
```



> Kotlin任何一个默认非抽象类均不能被继承
>



继承该open类,继承关键字变成了一个冒号



```kotlin
class Student : Person() {
    //TODO,Person()是构造函数
}
```



#### 主构造函数


> 主构造函数没有函数体,直接定义在类名的后面  
  
主构造函数只能有一个  
  
一个函数可以没有主构造函数(这种情况非常特殊,很少见,但是允许)
>



```kotlin
class Student(val sno: String, val grade: Int) : Person() {
    //TODO
}
```



进行实例化时,需要传入构造函数的所有参数



```kotlin
val student = Student("a123", 5)
```



如果显示的声明了父类的主构造函数体,那么在子类继承父类时需要传入相应的参数



```kotlin
open class Person(val name: String, val age: Int){
    //TODO
}
//继承Person
class Student(val sno: String, val grade: Int, name: String, age: Int) : Person(name, age){
    //TODO
}
```



#### 次构造函数


> 当一个类中同时有主/次构造函数时,次构造函数必须调用主构造函数  
  
次构造函数有函数体  
  
关键字是: constructor  
  
次构造函数可以有多个
>



```kotlin
class Student(val sno: String, val grade: Int, name: String, age: Int) : Person(name, age) {
    constructor(name: String, age: Int) : this("", 0, name, age) {
    }
    constructor() : this("", 0) {
    }
}
```



第一个次构造函数接收name和age两个参数,然后又通过this关键字调用了主构造函数,并将sno和grade两个参数赋初始值



第二个次构造函数不接收任何参数,**它通过this关键字调用了第一个次构造函数**,并将name和age参数赋初始值,这种调用方式是**间接调用主构造函数**



**注意**:需要关注的关键点是参数列表,第二个次构造函数this的参数列表对应的是第一个次构造函数



### 3.3 接口


接口与Java基本一致  
  
不同的时,kotlin中类的继承和接口的实现都统一用冒号



JDK1.8之后和kotlin支持接口的默认实现



```kotlin
interface Study {
    fun doHomework() {
        println("do homework default implementation.")
    }
    fun Study() {}
}
```



现在,当一个类去实现Study接口时,指挥强制要求实现readBook函数,而doHomework函数可以自由选择实现或者不实现.

| 修饰符 | Java | Kotlin |
| --- | --- | --- |
| public | 所有类可见 | 所有类可见(默认) |
| private | 当前类可见 | 当前类可见 |
| protected | 当前类,子类,同一包路径下的类可见 | 当前类,子类可见 |
| default | 同一包路径下的类可见(默认) | 无 |
| internal | 无 | 同一模块中的类可见 |




### 3.4 数据类与单例类


> MVC MVP MVVM 之类的架构模式,其中M指的就是数据类
>



#### 数据类


数据类通常需要重写equals(),hashCode(),toString()这几个方法  
  
其中equals()判断两个数据类是否相等.hashCode()方法作为equals()的配套方法也需要重写  
  
toString()可以让输出log更加清晰



在Kotlin中,只需要在数据类前声明关键字data,Kotlin就会根据主构造函数中的参数自动生成equals(),hashCode()toString()等方法



```kotlin
data class Cellphone(val brand: String, val price: Double)
//注意在参数列表添加需要的参数
```



#### 单例类


Java最常见的单例类



```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {};
    public synchronized static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    public void singletonTest() {
        System.out.println("singletonTest is called.");
    }
}
```



在Kotlin中,只需要在类前面声明object关键字,那么就可以将该类变为单例类.



```kotlin
object Singleton {
    fun singletonTest() {
        println("singletonTest is called.")
    }
}
```



## 4 Lambda编程


## 4.1 集合的创建和遍历


**List**  
  
创建一个集合



```kotlin
//正常创建一个集合并添加一些数据
val list = Arraylist<String>()
list.add("apple")
list.add("orange")
list.add("banana")

//kotlin提供了一个内置方法listOf()简化写法
val list = listOf("apple", "orange", "banana")
```



使用for-in进行循环遍历



```kotlin
fun main() {
    val list = listOf("apple", "orange", "banana")
    for (fruit in list) {
        println(fruit)
    }
}
```



**注意:** listOf()方法创建的是一个不可变的集合,无法对该集合进行添加,修改或者删除操作,只能读取.  
  
如果需要创建一个可变的集合,那么使用mutableListOf()方法就可以了.



```kotlin
val list = mutableListOf("apple", "orange", "banana")
```



**Set**



> Set集合与List集合类似,实际上就是将创建集合的方法换成了setOf()和mutableSetOf()方法  
  
Set集合的底层使用的是hash映射机制来存放数据,因此集合中的元素是无序的
>



**Map**  
  
Map是一种键值对形式的数据结构



```kotlin
//常规写法
val map = HashMap<String, Int>()
map.put("Apple", 1)
map.put("Banana", 2)

//Kotlin中建议的写法
map["Apple"] = 1
map["Banana"] = 2
map["Orange"] = 3
```



Kotlin提供了Map初始化方法mapOf()和mutableMapOf()



```kotlin
val map = mapOf("Apple" to 1, "Banana" to 2, "Orange" to 3)
//for-in 循环有所不一样
for ((fruit, number) in map) {
    println("fruit is " + fruit + " number is " + number)
}
```



## 4.2 集合的函数式API


先看一段代码



```kotlin
val list = listOf("apple", "orange", "banana")
val maxLengthFruit = list.maxBy { it.length }
println("maxLength fruit is: " + maxLengthFruit)
//好吧,看不懂......
```



> Lambda就是一小段可以作为参数传递的代码  
  
正常情况下,向某个函数传参数时只能传入变量,借助Lambda允许传入一小段代码
>



**Lambda语法结构**



```kotlin
{ 参数名1: 参数类型, 参数名2: 参数类型 -> 函数体 }
```



+ 最外层是一对大括号,如果有参数传入到lambda表达式中的话,还需要声明参数列表.
+ 参数列表的结尾使用一个"->"符号,表示参数列表的结束以及函数体的开始
+ 函数体通常不建议太长,但是Kotlin中并没有规定不太长是多长
+ **函数体最后一行代码会默认作为Lambda表达式的返回值**



再来看一段代码



```kotlin
val list = listOf("apple", "orange", "banana")
val lambda = { fruit: String -> fruit.length }
val maxLengthFruit = list.maxBy (lambda)
```



**接下来进行一系列眼花缭乱的操作:**



首先,不需要特别定义一个lambda变量



```kotlin
val maxLengthFruit = list.maxBy ( { fruit: String -> fruit.length } )
```



然后,Kotlin规定,当Lambda参数是函数的最后一个参数时,可以将Lambda表达式移到函数括号的外面



```kotlin
val maxLengthFruit = list.maxBy () { fruit: String -> fruit.length }
```



接下来,如果Lambda参数时函数的唯一一个参数的话,还可以将函数的括号省略掉



```kotlin
val maxLengthFruit = list.maxBy { fruit: String -> fruit.length }
```



最后,当Lambda表达式的参数列表只有一个参数时,也不必声明参数名称,直接使用it来代替



```kotlin
val maxLengthFruit = list.maxBy { it.length }
```



> 更新: 2022-07-22 11:27:47  
> 原文: <https://www.yuque.com/zhangxiaofani4cu/xih3ez/wbwguv>