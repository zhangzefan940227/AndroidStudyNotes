函数类型声明：（以往习惯的是类类型，这个是函数类型）

```kotlin
var func1: (String, Int) -> String
```

`(String, Int) -> String`表示接收形参为A，B，返回类型为C

函数类型初始化

```kotlin
val student: (String, Int) -> String = { name, age ->
    val student = Student(name, age)
    student.name + "|" + student.age
}
```

函数类型变量，可作为形参

```kotlin
fun test(student: (String, Int) -> String) {
    println(student("张三", 19))
}
```

函数类型起别名：

```kotlin
typealias MyStudent = (String, Int) -> String

fun main() {
    var student: MyStudent
}
```

变量可以直接表示这个函数

```kotlin
fun test(name: String, age: Int): String {
    return "666"
}

fun main() {
    var student:(String, Int) -> String = ::test //使用双冒号来引用一个现成的函数
}
```

上面的示例，变更一下， 改使用匿名函数

```kotlin
fun mian() {
    var test:(String, Int) ->String = fun(name: String, age: Int): String {
        return "666"
    }
}
```

上面的示例，简化为lambda表达式

```kotlin
var test2: (String) -> String = {
    println("默认参数为$it")
    "666"
} // 形参只有一个，默认名称为it

var test3: (String, Int) -> String = {name,age ->
    println("两个参数分别为$name 和$age")
    "666"
} // 形参有两个，可以分别指定名称

var test4: (String, Int) -> String = { name, _ ->
    println("指定的参数为$name")
    "666"
} //形参有两个，也可以不指定名称，用 _ 代替
```