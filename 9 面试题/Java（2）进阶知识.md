## Java 部分（二）高级知识点

### 1.  <span id="java_advance_1">哪些情况下的对象会被垃圾回收机制处理掉？</span>

垃圾回收机制中最基本的做法是分代回收。内存区域被划分为三代：新生代、老年代和永久代。对于不同世代可以使用不同的垃圾回收算法。一般来说，一个应用中的大部分对象的存活时间都很短，基于这一点，对于新生代的垃圾回收算法就可以很有针对性。

参考自：[Java — 垃圾收集灵魂拷问三连（一）](http://omooo.top/2017/12/25/Java%20---%20%E5%85%B3%E4%BA%8E%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E7%81%B5%E9%AD%82%E6%8B%B7%E9%97%AE%E4%B8%89%E8%BF%9E%EF%BC%88%E4%B8%80%EF%BC%89)

### 2. <span id="java_advance_2">讲一下常见的编码方式？</span>

编码的原因：

	计算机存储信息的最小单元是一个字节即八个bit，所以能表示的字符范围是0 - 225 个。然而要表示的符号太多了，无法用一个字节来完全表示，要解决这个矛盾必须需要一个新的数据结构char，从char到byte必须编码。编码方式规定了转化的规则，按照这个规则就可以让计算机正确的表示我们的字符。编码方式有以下几种：

ASCII码：

	总共有128个，用一个字节的低七位表示，0 - 31是控制字符，如回车键、换行等等。32 - 126是打印字符，可以通过键盘输入并且能够显示出来。

ISO-8859-1：

	ISO组织在ASCII码基础上又制定了一些列标准用来扩展ASCII编码，其中IS-8859-1涵盖了大多数西欧语言字符，单字节编码，总共表示256个字节，应用广泛。

GB2312、GBK：

	双字节编码，表示汉字。

UTF-16：

	UTF - 16具体定义了Unicode字符在计算机中的存取方法。UTF - 16用两个字节来表示Unicode转化格式，这个是定长的表示方法，不论什么字符都可以用两个字节表示，两个字节是十六个bit，所以叫UTF - 16.

UTF- 8：

UTF-16统一采用两个字节表示一个字符，虽然表示上非常简单方便，但是有很大一部分字符用一个字节就可以表示，显然是浪费了存储空间。于是UTF-8应运而生，UTF-8采用一种变长技术，每个编码区域有不同的字码长度，不同类型的字符可以是由1-6个字节组成。

参考自：[几种常见的编码格式](https://www.cnblogs.com/mlan/p/7823375.html)



### 3. <span id="java_advance_3">UTF-8编码中中文占几个字节，int型几个字节？</span>

中文占字节数可以是2、3和4个的，最高为4个字节。

参考自：[http://blog.csdn.net/hellokatewj/article/details/24325653](http://blog.csdn.net/hellokatewj/article/details/24325653)



### 4. <span id="java_advance_4">静态代理和动态代理的区别，什么场景使用？</span>

参考自：[https://www.jianshu.com/p/2f518a4a4c2b](https://www.jianshu.com/p/2f518a4a4c2b)



### 5. <span id="java_advance_5">Java的异常体系</span>

异常简介：异常是阻止当前方法继续执行的问题，如：文件找不到、网络连接失败、非法参数等等。发现异常的理想时期是编译阶段，然而编译阶段不能找出所有的异常，余下的问题必须在运行期间。

Java中的异常分为可查异常和不可查异常。

**可查异常：**

	即编译时异常，只编译器在编译时可以发现的错误，程序在运行时很容易出现的异常状况，这些异常可以预计，所以在编译阶段就必须手动进行捕捉处理，即要用try-catch语句捕获它，要么用throws子句声明抛出，否者编译无法通过，如：IOException、SQLException以及用户自定义Exception异常。

**不可查异常：**

	不可查异常包括**运行时异常**和**错误**，他们都是在程序运行时出现的。异常和错误的区别：异常能被程序本身处理，错误是无法处理的。**运行时异常指的是**程序在运行时才会出现的错误，由程序员自己分析代码决定是否用try...catch进行捕获处理。如空指针异常、类转换异常、数组下标越界异常等等。**错误是指**程序无法处理的错误，表示运行应用程序中较严重的问题，如系统崩溃、虚拟机错误、动态连接失败等等。大多数错误与代码编写者执行的操作无关，而表示代码运行时JVM出现的问题。

**Java异常类层次结构图：**

![](https://upload-images.jianshu.io/upload_images/81578-ce7c03b8b1930315.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

从上图可以看出Java通过API中Throwable类的众多子类描述各种不同的异常。因而，Java异常都是对象，是Throwable子类的实例。

**异常处理机制：**

当异常发生时，将使用new在堆上创建一个异常对象，对于这个异常对象，有两种处理方式：

1. 使用throw关键字将异常对象抛出，则当前执行路径被终止，异常处理机制将在其他地方寻找catch快对异常进行处理
2. 使用try...catch在当前逻辑中进行捕获处理

**throws 和 throw：**

throws：一个方法在声明时可以使用throws关键字声明可能会产生的若干异常

throw：抛出异常，并退出当前方法或作用域

使用 finally 清理：

在 Java 中的 finally 的存在并不是为了释放内存资源，因为 Java 有垃圾回收机制，因此需要 Java 释放的资源主要是：已经打开的文件或者网络连接等。

在 try 中无论有没有捕获异常，finally 都会被执行。

参考自：[https://www.jianshu.com/p/ffca876ce719](https://www.jianshu.com/p/ffca876ce719)



### 6. <span id="java_advance_6">谈谈你对解析与分派的认识</span>

[深入理解Java虚拟机之：多态性实现机制----静态分派与动态分配](https://www.jianshu.com/p/1976b01c07d2)



### 7. <span id="java_advance_7">修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法？</span>



### 8. <span id="java_advance_8">Java中实现多态的机制是什么？</span>



### 9. <span id="java_advance_9">如何将一个Java对象序列化到文件里？</span>



### 10. <span id="java_advance_10">说说你对Java反射的理解</span>

反射：在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。

关于反射的基本使用可看：[Java-反射的理解与使用-(原创)](https://www.jianshu.com/p/2302ad5dbfad)

**反射是一种具有与类进行动态交互能力的一种机制**。

**反射的作用：**

在 Java 和 Android 开发中，一般情况下下面几种场景会用到反射机制：

* 需要访问隐藏属性或者调用方法改变程序原来的逻辑，这个在开发中是很常见的，由于一些原因，系统并没有开放一些接口出来，这个时候利用反射是一个有效的解决办法。
* 自定义注解，注解就是在运行时利用反射机制来获取的。
* 在开发中动态加载类，比如在 Android 中的动态加载解决65k问题等等，模块化和插件化都离不开反射。

**反射的工作原理**

我们知道，每个 Java 文件最终都会被编译成一个 .class 文件，这些 Class 对象承载了这个类的所有信息，包括父类、接口、构造函数、方法、属性等等，这些class文件在程序运行时会被 ClassLoader 加载到虚拟机中。当一个类被加载以后，Java 虚拟机就会在内存中自动产生一个 Class 对象，而一般情况下用 new 来创建对象，实际上本质都是一样的，只是底层原理对我们开发者透明罢了。有了 class 对象的引用，就相当于有了 Method、Field、Constructor 的一切信息，在 Java 中，有了对象的引用就有了一切，剩下就靠自己发挥了。

参考自：[Java反射以及在Android中的特殊应用](https://www.jianshu.com/p/8f394d90a85c)



### 11.<span id="java_advance_11"> 说说你对Java注解的理解</span>

注解（Annotion）是一个接口（前加 @），程序可以通过反射来获取指定程序元素的Annotion对象，然后通过Annotion对象来获取注解里面的元数据。

注解可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。从某些方面看，Annotion就像修饰符一样被使用，并应用于包、类型、构造方法、方法、成员变量、参数、本地变量的声明中。

Annotation 的行为十分类似 public、final 这样的修饰符。每个 Annotation 具有一个名字和成员，每个 Annotation 的成员具有被称为 name=value 对的名字和值，name=value 装载了 Annotation 的信息，也就是说注解中可以不存在成员。

使用注解的基本规则：Annotation 不能影响程序代码的执行，无论增加、删除 Annotation，代码都始终如一的执行。

关于注解的基本使用可见：[https://www.jianshu.com/p/91393daaaf32](https://www.jianshu.com/p/91393daaaf32)

参考自：[https://www.jianshu.com/p/8da24b7cf443](https://www.jianshu.com/p/8da24b7cf443)

从JDK5开始，Java增加了Annotation，注解是代码里的特殊标记，这些标记可以在编译、类加载、运行时被读取，并执行相应的处理。通过使用Annotation，开发人员可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充的信息。

Annotation提供了一种为程序元素（包、类、构造器、方法、成员变量、参数、局部变量）设置元数据的方法。Annotation不能运行，**它只有成员变量，没有方法**。Annotation跟public、final修饰符一样，都是程序元素的一部分，Annotation不能作为一个程序元素使用。

#### 1. 定义Annotation

定义新的Annotation类型使用@interface关键字（在原有interface关键字前增加@符合），例如：

```java
//定义注解
public @interface Test{
}
//使用注解
@Test
public class MyClass{
....
}
```

**1.1 成员变量**

Annotation只有成员变量，没有方法。Annotation的成员变量在定义中以“无形参的方法”形式声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。例如：

```java
//定义注解
public @interface MyTag{
    string name();
    int age();
  	string id() default 0;
}
//使用注解
public class Test{
    @MyTag(name="红薯"，age=30)
    public void info(){
    ......
    }
}
```

一旦在Annotation里定义了成员变量后，使用该Annotation时就应该为该Annotation的成员变量指定值。

也可以在定义Annotation的成员变量时，为其指定默认值，指定成员变量默认值使用default关键字。

根据Annotation是否包含了成员变量，可以把Annotation分为以下两类：

* 标记Annotation

  没有成员变量的Annotation被称为标记，这种Annotation仅用自身的存在与否来为我们提供信息，例如@override等

* 元数据Annotation

  包含成员变量的Annotation。因为它可以接收更多的元数据，因此被称为元数据Annotation。

**1.2 元注解**

在定义Annotation时，也可以使用JDK提供的元注解来修饰Annotation定义。JDK提供了如下四个元注解（注解的注解，不是上诉的“元数据”）

* @Retention
* @Target
* @Documented
* @Inherited

**1.2.1 @Retention**

用于指定Annotation可以保留多长时间。

@Retention包含一个名为“value”的成员变量，该value成员变量是RetentionPolicy枚举类型。使用@Retention时，必须为其value指定值，value成员变量的值只能是如下三个：

* RetentionPolicy.SOURCE

  Annotation只保留在源代码中，编译器编译时，直接丢弃这种Annotation

* RetentionPolicy.CLASS

  编译器会把Annotation记录在class文件中，当运行Java程序时，JVM中不再保留该Annotation

* RetentionPolicy.RUNTIME

  编译器把Annotation记录在class文件中，当运行Java程序时，JVM会保留该Annotation，程序可以通过反射获取该Annotation的信息。

```
//name=value形式
//@Retention(value=RetentionPolicy.RUNTIME)

//直接指定
@Retention(RetentionPolicy.RUNTIME)
public @interface MyTag{
	String name() default "我兰";
}
```

**1.2.2 @Target**

@Target指定Annotation用于修饰哪些程序元素。@Target也包含一个名为“value”的成员变量，该value成员变量类型为ElementType[]，同样为枚举类型，值有以下几个：

* ElementType.TYPE   能修饰类、接口或枚举类型
* ElementType.FIELD   能修饰成员变量
* ElementType.METHOD   能修饰方法
* ElementType.PARAMETER   能修饰参数
* ElementType.CONSTRUCTOR   能修饰构造器
* ElementType.LOCAL_VARIABLE   能够修饰局部变量
* ElementType.ANNOTATION_TYPE   能修饰注解
* ElementType.PACKAGE   能修饰包

```java
//单个ElementType
@Target(ElementType.FIELD)
public @interface AnnTest {
	String name() default "sunchp";
}
//多个ElementType
@Target({ ElementType.FIELD, ElementType.METHOD })
public @interface AnnTest {
	String name() default "sunchp";
}
```

**1.2.3 @Documented**

如果定义注解时，使用了@Documented修饰定义，则在用javadoc命令生成API文档后，所有使用该注解修饰的程序元素，将会包含该注解的说明。

**1.2.4 @Inherited**

指定Annotation具有继承性。



**1.3 基本Annotation**

* @Override

  限定重写父类方法。对于子类中被@Override修饰的方法，如果存在对应的被重写的父类方法，则正确；如果不存在，则报错。@Override只能作用于方法，不能作用于其他程序元素。

* @Deprecated

  用于表示某个程序元素（类、方法等等）已过时，如果使用被@Deprecated修饰的类或方法等，编译器会发出警号。

* @SuppressWarning

  抑制编译器警号。指示被@SuppressWarning修饰的程序元素（以及该程序元素中的所有子元素，例如类以及该类中的方法...）取消显示指定的编译器警告，例如，常见的@SuppressWarning (value="unchecked")

* @SafeVarargs

  告诉编译器忽略可变长度参数可能引起的类型转换问题，该注解修饰的方法必须为 static。

#### 2. 提取Annotation信息（反射）

当开发者使用了Annotation修饰了类、方法、Field等成员之后，这些Annotation不会自己生效，必须由开发者提供相应的代码来提取并处理Annotation信息，这些处理和提取Annotation的代码统称为APT（Annotation Processing Tool）。

JDK主要提供了两个类，来完成Annotation的提取：

* java.lang.annotation.Annotation

  这个接口是所有Annotation类型的父接口

* java.lang.reflect.AnnotatedElement

  该接口代表程序中被注解的程序元素

**2.1 java.lang.annotation.Annotation**

该接口源码：

```java
package java.lang.annotation;

public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```

其中主要方法是annotationType()，用于返回该注解的java.lang.Class

**2.2 java.lang.reflect.AnnotatedElement**

接口源码：

```java
package java.lang.reflect;

import java.lang.annotation.Annotation;

public interface AnnotatedElement {

  	//判断该程序元素上是否存在指定类型的注解，如果存在返回true，否则false
    boolean isAnnotationPresent(Class<? extends Annotation> annotationClass);
  
	//返回该程序元素上存在的指定类型的注解，如果该类型的注解不存在，则返回null
    <T extends Annotation> T getAnnotation(Class<T> annotationClass);

  	//返回该程序元素上存在的所有注解
    Annotation[] getAnnotations();

    Annotation[] getDeclaredAnnotations();
}
```

AnnotatedElement接口是所有程序元素（例如java.lang.Class、java.lang.reflect.Method、java.lang.reflect.Constructor等）的父接口。所以程序通过反射获取某个类的AnnotatedElement对象后，就可以调用该对象的isAnnotationPresent()、getAnnotation()等方法来访问注解信息。

**为了获取注解信息，必须使用反射知识。**

注：如果想要在运行时获取注解信息，在定义注解的时候，该注解必须要使用@Retention(RetentionPolicy.RUNTIME)修饰。

**2.3 示例**

**2.3.1 标记Annotation**

```java
//定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyTag {

}
//注解处理
class ProcessTool {
    static void process(String clazz) {
        Class targetClass = null;
        try {
            targetClass = Class.forName(clazz);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        assert targetClass != null;
        for (Method method : targetClass.getMethods()) {
            if (method.isAnnotationPresent(MyTag.class)) {
                System.out.println("被MyTag注解修饰的方法名：" + method.getName());
            } else {
                System.out.println("没有被MyTag注解修饰的方法名：" + method.getName());
            }
        }
    }
}
//测试类
public class TagTest {

    @MyTag
    public static void m1() {

    }

    public static void m2() {

    }

    public static void main(String[] args) {
        ProcessTool.process("annotation.TagTest");
    }
}
//输出
没有被MyTag注解修饰的方法名：main
被MyTag注解修饰的方法名：m1
没有被MyTag注解修饰的方法名：m2
没有被MyTag注解修饰的方法名：wait
没有被MyTag注解修饰的方法名：wait
没有被MyTag注解修饰的方法名：wait
没有被MyTag注解修饰的方法名：equals
没有被MyTag注解修饰的方法名：toString
没有被MyTag注解修饰的方法名：hashCode
没有被MyTag注解修饰的方法名：getClass
没有被MyTag注解修饰的方法名：notify
没有被MyTag注解修饰的方法名：notifyAll
```

**2.3.2 元数据Annotation**

```java
//定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyTag {
    String name() default "Omooo";

    int age() default 21;
}
//注解处理
class ProcessTool {
    static void process(String clazz) {
        Class targetClass = null;
        try {
            targetClass = Class.forName(clazz);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        assert targetClass != null;
        for (Method method : targetClass.getMethods()) {
            if (method.isAnnotationPresent(MyTag.class)) {
                MyTag tag = method.getAnnotation(MyTag.class);
                System.out.println("方法：" + method.getName() + " 的注解内容为：" + tag.name() + "  " + tag.age());
            }
        }
    }
}
//测试类
public class TagTest {

    @MyTag
    public static void m1() {

    }

    @MyTag(name = "当当猫", age = 20)
    public static void m2() {

    }

    public static void main(String[] args) {
        ProcessTool.process("annotation.TagTest");
    }
}
//输出
方法：m1 的注解内容为：Omooo  21
方法：m2 的注解内容为：当当猫  20
```

#### 3. 注解本质

* 注解实质上会被编译器编译为接口，并且继承java.lang.annotation.Annotation接口
* 注解的成员变量会被编译器编译成同名的抽象方法
* 根据Java的class文件规范，class文件中会在程序元素的属性位置记录注解信息。

#### 4. 注解的意义

1. 为编译器提供辅助信息

   Annotation可以为编译器提供额外的信息，以便于检测错误，抑制警告等。

2. 编译源代码时进行额外操作

   软件工具可以通过处理Annotation信息来生成源代码，xml文件等等。

3. 运行时处理

   有一些Annotation甚至可以在程序运行时被检测、使用。


总之，注解是一种元数据，起到了“描述、配置”的作用。


引自：[http://www.open-open.com/lib/view/open1423558996951.html](http://www.open-open.com/lib/view/open1423558996951.html)

### 12. <span id="java_advance_12">说说你对依赖注入的理解</span>

依赖注入：可以通过这个服务来安全的注入组件到应用程序中，在应用程序部署的时候还可以选择从特定的接口属性进行注入。

参考自：

[用Dagger2在Android中实现依赖注入](https://www.jianshu.com/p/f713dd40e037)

[反射、注解与依赖注入总结](https://www.jianshu.com/p/24820bf3df5c)



### 13. <span id="java_advance_13">说一下泛型原理，并举例说明</span>



### 14.<span id="java_advance_14"> Java中String的了解</span>



### 15. <span id="java_advance_15">String为什么要设计成不可变的？</span>

String 是 Java 中一个不可变类，所以他一旦被实例化就无法被修改。为什么要把String设计成不可变的呢？那就从内存、同步和数据结构层面上谈起。

**字符串池**

字符串池是方法区中的一部分特殊存储。当一个字符串被创建的时候，首先会去这个字符串池中查找。如果找到，直接返回对该字符串的引用。

下面的代码只会在堆中创建一个字符串：

```java
String string1 = "abcd";
String string2 = "abcd";
```

![](http://www.hollischuang.com/wp-content/uploads/2016/03/QQ20160302-3.png)

**如果字符串可变的话，当两个引用指向同一个字符串时，对其中一个做修改就会影响另外一个。**

```java
String str= "123";
str = "456";
```

执行过程如下：

![](https://upload-images.jianshu.io/upload_images/2466095-96d3d70753c70c1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

执行第一行代码时，在堆上新建一个对象实例 123，str 是一个指向该实例的引用，**引用包含的仅仅只是实例在堆上的内存地址而已**。执行第二行代码时，仅仅只是改变了 str 这个引用的地址，指向了另外一个实例 456。**给 String 赋值仅仅只是改变了它的引用而已，并不会真正的去改变它本来内存地址上的值**。这样的好处也是显而易见的，最简单的当存在多个 String 的引用指向同一个内存地址时，改变其中一个引用的值并不会其他引用的值造成影响。

那如果我们这样写呢：

```java
  String str1 = "123";
  String str2 = new String("123");
  System.out.println(str1 == str2);
```

结果是 false。JVM为了字符串的复用，减少字符串对象的重复创建，特别维护了一个字符串常量池。第一种字面量形式的写法，会直接在字符串常量池中查找是否存在值 123，若存在直接返回这个值的引用，若不存在则创建一个值123的String对象并存入字符串常量池中。而使用new关键字，则会直接才堆上生成一个新的String对象，并不会理会常量池中是否有这个值。所以本质上 str1 和 str2 指向的内存地址是不一样的。

那么，使用 new 关键字生成的 String 对象可以进入字符串常量池吗？答案是肯定的，String 类提供了一个 native 方法 intern() 用来将这个对象加入字符串常量池：

```java
  String str1 = "123";
  String str2 = new String("123");
  str2=str2.intern();
  System.out.println(str1 == str2);
```

结果为 true。str2 调用 intern() 方法后，首先会在字符串常量池中寻找是否存在值为 123 的对象，若存在直接返回该对象的引用，若不存在，加入 str2 并返回。上诉代码中，常量池中已经存在了值为 123 的 str1 对象，则直接返回 str1 的引用地址，使得 str1 和 str2 指向同一个内存地址。

**缓存 Hashcode**

Java 中经常会用到字符串的哈希码。例如，在HashMap中，字符串的不可变能保证其Hashcode永远保持一致，避免了一些不必要的麻烦。这也就意味着每次使用一个字符串的hashcode的时候不用重新计算一次，更加高效。

**安全性**

String被广泛的使用在其他类中充当参数，如果字符串可变，那么类似操作就会可能导致安全问题。可变的字符串也可能导致反射的安全问题，因为它的参数也是字符串。

**不可变对象天生就是线程安全的**

因为不可变对象不能被改变，所以他们可以自由的在多个线程之间共享，不需要做任何同步处理。

总之，String被设计成不可变的主要目的是为了安全和高效。当然，缺点就是要创建多余的对象而并非改变其值。

参考自：

[https://www.jianshu.com/p/b1d62928552d](https://www.jianshu.com/p/b1d62928552d)

[http://www.hollischuang.com/archives/1246](http://www.hollischuang.com/archives/1246)



### 16.<span id="java_advance_16"> Object类的equal和hashCode方法重写，为什么？</span>



### 17. <span id="java_advance_17">为什么Java里的匿名内部类只能访问final修饰的外部变量？</span>

	因为匿名内部类最终会被编译成一个单独的类，而被该类使用的变量会以构造函数参数的形式传递给该类。如果变量不定义为final的，参数在匿名内部类中可以被修改，进而造成和外部的变量不一致的问题，为了避免这种不一致的情况，规定匿名内部类只能访问final修饰的外部变量。



### 18. <span id="java_advance_18">Synchronized</span>

	synchronized，是Java中用于解决并发情况下数据同步访问的一个很重要的关键字。当我们想要保证一个共享资源在同一个时间只会被一个线程访问到时，我们可以在代码中使用synchronized关键字对类或者对象加锁。

在Java中，synchronized有两种使用方式，同步方法和同步代码块。

对于同步方法，JVM采用ACC_SYNCHRONIZED标记符来实现同步。对于同步代码块，JVM采用monitorenter、monitorexit 两个指令来实现同步。

**同步方法**
方法级的同步是隐式的，同步方法的常量池中会有一个ACC_SYNCHRNZED 标志，当某个线程要访问某个方法的时候，会检查是否有 ACC_SYNCHORIZED ，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后在释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果方法在执行过程中发生了异常，并且方法内部并没有处理该异常，那么异常被抛到方法外面之前监视器锁会被自动释放。

**同步代码块**
同步代码块使用monitorenter和monitorexit两个指令实现。可以把执行monitorenter指令理解为加锁，执行monitorexit理解为释放锁。每个对象维护着一个记录被锁次数的计数器，未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为1，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行monitorexit指令）的时候，计数器在自减。当计数器为0的时候，锁将被释放，其他线程便可以获得锁。

**synchronized与原子性**
原子性是指一个操作是不可中断的，要全部执行完成，要不就都不执行。

线程是CPU调度的基本单位，CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间轮换，就会发生原子性问题。

在Java中，为了保证原子性，提供了两个高级的字节码指令 monitorenter 和 monitorexit 。前面介绍过，这两个字节码指令，在Java中对应的关键字就是 synchronized。

通过 monitorenter 和 monitorexit 指令，可以保证被 synchronized修饰的代码在同一时间只能被一个线程访问，在锁未释放之前，无法被其他线程访问到。因此，在Java中可以使用 synchronized 来保证方法和代码块内的操作是原子性的。

线程一在执行monitorenter指令的时候，会对Monitor进行加锁，加锁后其他线程无法获得锁，除非线程一主动解锁。即使在执行过程中，由于某种原因，比如CPU时间片用完，线程一放弃了CPU，但是，他并没有进行解锁，而由于 synchorized 的锁是可以重入的，下一个时间片还是只能被他自己获取到，还是会继续执行代码，直到所有代码执行完，这就保证了原子性。

**synchroized与可见性**
可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之后也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主内存之间进行数据同步进行。所以，就可能出现线程一修改了某个变量的值，但线程二不可见的情况。

前面我们介绍过，被 synchronized 修饰的代码，在开始执行时会加锁，执行完成后会进行解锁。而为了保证可见性，有一条规则是专业的：对一个变量解锁之前，必须先把变量同步到主内存中。这样解锁后，后续线程就可以访问到被修改后的值。

所以，被 synchronized 关键字锁住的对象，其值是具有可见性的。

**synchronized与有序性**
有序性即程序执行的顺序按照代码的先后顺序执行。

除了引入了时间片以外，由于处理器优化和指令重排，CPU还可能对输入代码进行乱序执行，这就可能存在有序性问题。

这里需要注意的是，synchronized 是无法禁止指令重排和处理器优化的，也就是说，synchronized 无法避免上述提到的问题。那么为什么还说synchronized 也提供了有序性保证呢？

这就要把有序性的概念扩展一下了，Java程序中天然的有序性可以总结为一句话：如果在本线程内观察，所有操作都是天然有序的，如果在一个线程中观察另外一个线程，所有的操作都是无序的。

以上这句话也是《深入理解Java虚拟机》中的原句，但是怎么理解呢？这其实和 as-if-serial语义有关。

as-if-serial 语义的意思是指：不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果都不能被改变，编译器和处理器无论如何优化，都必须遵守 as-if-serial语义。

简单来说，as-if-serial语义保证了单线程中，指令重排是有一定限制的，而只要编译器和处理器都遵守这个语义，那么就可以认为单线程程序是按照顺序执行的，当然，实际上还是有重排，只不过我们无需关心这种重排的干扰。

所以说，由于synchronized修饰的代码，同一时间只能被同一个线程访问，那么也就是单线程执行，所以可以保证其有序性。

**synchronized与锁优化**

无论是ACC_SYNCHORIZED还是monitorenter、monitorexit都是基于Monitor实现的，在Java虚拟机（HotSpot）中，Monitor是基于C++实现的，由ObjectMonitor实现。

ObjectMonitor类中提供了几个方法，如 enter、exit、wait、notify、notifyAll 等。sychronized 加锁的原理，会调用 objectMonitor的enter方法，解锁的时候会调用 exit 方法。事实上，只有在JDK1.6之前，synchronized的实现才会直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁。为什么说这种方式操作锁很重呢？

Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就是要从用户态转换为核心态，因此状态转换需要花费很多的处理器时间，对于代码简单的同步块，状态转换消耗的时间有可能比用户代码执行的时间还要长，所以说synchroized是java语言中一个重量级的操纵。

所以，在JDK1.6中出现对锁进行了很多的优化，进而出现了轻量级锁、偏向锁、锁消除，适应性自旋锁等等，这些操作都是为了在线程之间更高效的共享数据，解决竞争问题。

### 19.<span id="java_advance_19"> volatile</span>

**volatile用法**
volatile通常被比喻成轻量级的synchronized，也是Java并发编程中比较重要的一个关键字，和synchronized不同，volatile是一个变量修饰符，只能用来修饰变量，无法修饰方法以及代码块。

volatile的用法比较简单，只需要在声明一个可能被多线程同时访问的变量时，使用volatile修饰就可以了。

**volatile原理**
为了提高处理器的执行速度，在处理器和内存之间增加了多级缓存来提升，但是由于引入了多级缓存，就存在缓存数据不一致的问题。

但是，对于volatile变量，当对volatile变量进行写操作的时候，JVM会向处理器发送一条Lock前缀的指令，将这个缓存中的变量回写到系统主存中。

但是就算回写内存，如果其他处理器缓存的值还是旧的，在执行计算操作就会有问题，所以在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议。

**缓存一致性协议：**

每个处理器通过嗅探在总线上传播的数据来检测自己缓存的信息是不是过期了，当处理器发现自己缓存行对应的内存地址被修改了，就会将当前处理器的缓存行设置为无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。

所以，如果一个变量被volatile所修饰的话，在每次数据变化之后，其值都会被强制刷新入主存。而其他处理器的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中，这就保证了一个volatile修饰的变量在多个缓存中是可见的。

**volatile与可见性**
可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程也能够立即看到修改的值。

前面在关于volatile原理的时候讲过，Java中的volatile关键字提供了一个功能，那就是被修饰的变量在被修改后可以立即同步到主存中，被其修饰的变量在每次使用之前都是从主内存中刷新。因此，可以使用volatile来保证多线程操作时变量的可见性。

**volatile与有序性**
volatile禁止指令重排优化，这就保证了代码的程序会严格按照代码的先后顺序执行，这就保证了有序性。

**volatile与原子性**
在上面介绍synchronized的时候，提到过，为了保证原子性，需要通过字节码指令monitorenter和monitorexit，但是volatile和这两个指令没有任何关系。

**所以，volatile是不能保证原子性的。**

在以下两个场景中可以使用volatile替代synchronized：

1. 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程会修改变量的值
2. 变量不需要与其他状态变量共同参与不变约束

除了以上场景，都需要使用其他方式来保证原子性，如synchronized或者concurrent包。

synchronized可以保证原子性、有序性和可见性，而volatile只能保证有序性和可见性。



### 20. <span id="java_advance_20">类加载流程双亲委托机制</span>

**类加载流程**

1. 装载
2. 链接
   1. 验证
   2. 准备
   3. 解析
3. 初始化

**双亲委托机制**

某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才会自己去加载。

使用双亲委托模型的好处在于Java类随着它的类加载器一起具备类一种带有优先级的层次关系。例如类java.lang.Object，它存在rt.jar中，无论哪一个类加载器要加载这个类，最终都会委托处于模型最顶端的Bootstrap ClassLoader进行加载，因此Object类在程序的各类加载器环境中都是同一个类。相反，如果没有双亲委派模型，而是由各个类加载器自行加载的话，如果用户编写了一个java.lang.Object的同名类并放在ClassPath中，那么系统中将会出现多个不同的Object类，程序将变得混乱。因此，如果开发者尝试编写一个与rt.jar类库中重名的Java类，可以正常编译，但是永远无法被加载运行。

1. 当前ClassLoader首先从自己已经加载的类中查询是否此类已经记载，如果已经加载则可以直接返回已经加载的类。

   每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，等下次加载的时候就可以直接返回了。

2. 当前ClassLoader的缓存中没有找到被加载的类的时候，会委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到BootStrap ClassLoader。

3. 当所有的父类加载器都没有加载的时候，再有当前的类加载器加载，并将其放入它自己的缓存中，以便下次加载请求时直接返回。

为什么需要这样的委托机制呢？理解这个问题，我们要引入另外一个关于ClassLoader的概念“命名空间”，它是指要确定某一个类，需要类的全限定名以及加载此类的ClassLoader来共同确定。也就是说，即使两个类的全限定名相同，但是因为不同的ClassLoader加载了此类，那么在JVM中它是不同的类。明白了命名空间以后，我们再来看看委托模型。采用了委托模型以后加大了不同的ClassLoader的交互能力，比如上面说的，我们JDK本身提供的类库，比如HashMap、LinkedList等等，这些类由bootstrap类加载器加载以后，无论你程序中有多少个类加载器，那么这些类其实都是可以共享的，这样就避免了不同的类加载器加载了同样名字的不同类以后造成混乱。
