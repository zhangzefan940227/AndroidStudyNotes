# 1. <span id="java_base_1">Java中 == 和 equals 和 hashCode 的区别</span>
Java中的数据类型可分为两类，**基本数据类型**和**引用类型**。
- **基本数据类型**：byte、short、char、int、long、float、double、boolean。
- **引用类型**：类、接口、数组。

**==**

- **基本数据类型**：用双等号（==）比较的是**值**。
- **引用类型**：用双等号（==）比较的是他们在**内存中的存放地址**。对象是放在堆中的，栈中存放的是对象的引用（地址）。由此可见，**双等号是对栈中的值进行比较的**。如果要比较堆中对象是否相同，那么就要重写equals方法了。

**equals**

- Java中不可以使用equals对比基本数据类型。
- **引用类型**：默认情况下调用Object类的equals方法，主要是用于判断**对象的内存地址是不是相等**（即是不是指向同一个对象）。下面是Object类中的equals方法：

```java
public boolean equals(Object obj) {  
    return (this == obj);  
} 
```

Object类中的equals方法与==是等效的。

**但是**，要是类中覆盖了equals方法，那么就要根据具体代码来确定equals方法的作用了。**覆盖后的一般都是通过对象的内容是否相等来判断对象是否相等。** 

在 Object#equals 方法注释上，也给出了重写 equals 函数要遵守的规则：自反性、对称性、传递性和一致性，并且给出了具体示例。注释中还给出了，重写 equals 方法时也要重写 hashCode 方法，从而维持 hashCode 的语义，即如果对象相等，那么他们的哈希值必须相同。

**hashCode**

hashCode()方法返回的就是一个数值，从方法名上来看，其目的就是生成一个hash码，hash码的主要用途就是在**对对象进行散列的时候作为key输入**。

参考自：[http://blog.csdn.net/hla199106/article/details/46907725](http://blog.csdn.net/hla199106/article/details/46907725)

**补充：重写 equals 时为什么一定要重写 hashCode？**
首先我们要明确equals和hashCode的关系：
1. hashCode相等，equals不一定相等，两个类也不一定相同。
2. equals相等，则hashCode一定相等。

我们从下面这个例子入手：
```java
import java.util.HashSet;  
import java.util.Set;  
public class HashCodeExample {  
    public static void main(String[] args) {  
        Set<String> set = new HashSet();  
        set.add("Java");  
        set.add("Java");  
        set.add("MySQL");  
        set.add("MySQL");  
        set.add("Redis");  
        System.out.println("Set 集合长度:" + set.size());  
        System.out.println();  
        // 打印 Set 中的所有元素  
        set.forEach(d -> System.out.println(d));  
    } 
 }
----------------------------------------
程序输出结果：
Set 集合长度:3

Java
MySQL
Redis
```
从上述结果可以看出，重复的数据已经被 Set 集合“合并”了，这也是 Set 集合最大的特点：去重。

上文提到覆盖后的equals方法一般是通过对象的内容是否相等来判断对象是否相等的。如果我们只重写equals方法，那么就会出现下面这种情况。如下代码所示：

```java
public class EqualsExample {  
    public static void main(String[] args) {  
        // 对象 1  
        Persion p1 = new Persion();  
        p1.setName("Java");  
        p1.setAge(18);  
        // 对象 2  
        Persion p2 = new Persion();  
        p2.setName("Java");  
        p2.setAge(18);  
        // 创建 Set 集合  
        Set<Persion> set = new HashSet<Persion>();  
        set.add(p1);  
        set.add(p2);  
        // 打印 Set 中的所有数据  
        set.forEach(p -> {  
            System.out.println(p);  
        });  
    }  
}  
class Persion {  
    private String name;  
    private int age;  
    // 只重写了 equals 方法  
    @Override  
    public boolean equals(Object o) {  
        if (this == o) return true; // 引用相等返回 true  
        // 如果等于 null，或者对象类型不同返回 false  
        if (o == null || getClass() != o.getClass()) return false;  
        // 强转为自定义 Persion 类型  
        Persion persion = (Persion) o;  
        // 如果 age 和 name 都相等，就返回 true  
        return age == persion.age &&  
                Objects.equals(name, persion.name);  
    }  
    // 省略get set toString等方法
}
----------------------------------------
程序输出结果：
Persion{name='Java', age=18}
Persion{name='Java', age=18}
```
在上述代码中，我们重写equals方法，只要age name都相等，就返回true，那么在我们自定义的语义中，p1和p2是同一个对象。但是从上面的输出结果来看，Set中没有对p1和p2去重，即他们并不是相等。这就和我们之前提到的equals和hashCode关系的第2条相悖，因此必须重写hashCode，确保语义上的一致性。

# 2. <span id="java_base_2">int、char、long 各占多少字节数</span>

| 类型     | 位数   | 字节数  |
| ------ | ---- | ---- |
| boolen | 8    | 1    |
| int    | 32   | 4    |
| float  | 32   | 4    |
| double | 64   | 8    |
| char   | 16   | 2    |
| byte   | 8    | 1    |
| short  | 16   | 2    |
| long   | 64   | 8    |



# 3. <span id="java_base_3">int 和 Integer 的区别</span>

Java 为每一个基本数据类型都引入了对应的包装类，从Java 5 开始引入了自动装箱 / 拆箱机制，使得两者可以相互转化。**所以最基本的一点区别就是**：Integer 是int的包装类，int的初始值为零，而Integer的初值为null。int与Integer相比，会把Integer自动拆箱为int再去比。

参考自：[http://blog.csdn.net/login_sonata/article/details/71001851](http://blog.csdn.net/login_sonata/article/details/71001851)



# 4.  <span id="java_base_4">谈谈对Java多态的理解</span>

**多态即事物在运行过程中存在不同的状态。** 

多态的存在有三个前提
1. 要有继承关系，子类可以继承父类的方法和属性。
2. 子类重写父类方法，如果子类重写了父类的方法，则当用父类类型引用子类对象并调用该方法时，会执行子类的方法，而不是父类的方法。这就是多态的基础表现。
3. 父类数据类型的引用指向子类对象

弊端就是：不能使用子类特有的成员属性和子类特有的成员方法。如果非要用到不可，可以强制类型转换。

参考自：[https://item.btime.com/m_2s22uri6z16](https://item.btime.com/m_2s22uri6z16)



# 5.  <span id="java_base_5">String、StringBuffer、StringBuilder的区别</span>

- String：字符串常量，使用字符串拼接时是不同的两个空间。
- StringBuffer：字符串变量，线程安全，字符串拼接直接在字符串后追加。
- StringBuilder：字符串变量，非线程安全，字符串拼接直接在字符串后追加。

**执行效率**：StringBuilder > StringBuffer > String

String是一个常量，是不可变的，所以对于每次 += 赋值都会创建一个新的对象。

StringBuilder和StringBuffer都是可变的，当进行字符串拼接的时候采用append方法，在原来的基础上追加，所以性能要比String高，因为StringBuffer是线程安全的而StringBuilder是线程非安全的，所以StringBuilder的效率高于StringBuffer。

对于大数据量的字符串拼接，采用StringBuffer、StringBuilder。另一种说法，JDK1.6做了优化，通过String声明的字符串在进行用+进行拼接时，底层调用的是StringBuffer，所以性能基本上和后两者没什么区别。

# 6. <span id="java_base_6"> 什么是内部类？内部类的作用</span>

Java 常见的内部类有四种：成员内部类、静态内部类、方法内部类和匿名内部类。

**内部类的作用**：
- **内部类可以很好的实现隐蔽。** 一般的非内部类，是不允许有private和protectd等权限的，但内部类（除方法内部类）可以通过这些修饰符来实现隐蔽。
- **可以访问外围类元素。** 内部类拥有外部类的访问权限（分静态与非静态情况），通过这一特性可以比较好的处理类之间的关联性，将一类事物的流程放在一起内部处理。
- **通过内部类可以实现多重继承。** Java默认是单继承的，我们可以通过多个内部类继承实现多个父类，接着由于外部类完全可访问内部类，所以就实现了类似多继承的效果。
- **通过匿名内部类来优化简单的接口实现。**

参考自：[https://www.jianshu.com/p/7fe3af7f0f2d](https://www.jianshu.com/p/7fe3af7f0f2d)



# 7.  <span id="java_base_7">抽象类和接口的区别</span>

抽象类使用abstract class 定义，抽象类既可以有抽象方法也可以有其他类型的方法，既可以有静态属性也可以有非静态属性。

接口使用interface定义，属性用public final static 修饰，如果没有写关键字，虚拟机默认会加上关键字。JDK8之后接口里的方法既可以有抽象方法也可以有默认方法。

抽象类是一种类别，具有构造方法。接口是一种行为规范，没有构造方法。抽象类中可以没有抽象方法，他有自己的构造方法但是不能直接实例化对象，所以必须被子类继承之后才有意义。

抽象类只能单继承，接口可以被多个类实现，接口和接口之间可以多继承。

抽象类可以由默认的方法实现，接口根本不存在方法的实现。实现抽象类使用extends关键字来继承抽象类，如果子类不是抽象类，它需要提供抽象类中所有抽象方法的实现。子类使用关键字implements来实现接口，它需要提供接口中所有声明的方法的实现。

抽象方法可以有public、protectd 和 default 这些修饰符，而接口方法默认修饰符是public，不可以使用其他修饰符。

抽象类比接口速度快，接口是稍微有点慢的，这是因为它需要时间去寻找类中实现的方法。

参考自：[http://www.importnew.com/12399.html](http://www.importnew.com/12399.html)



# 8.  <span id="java_base_8">抽象类的意义</span>

为子类提供一个公共的类型；封装子类中重复的内容；定义有抽象方法，子类虽然有不同的实现，但该方法的定义是一致的；



# 9.  <span id="java_base_9">抽象类与接口的应用场景</span>

- 如果你拥有一些方法并且想让它们中的一些默认实现，那么就用抽象类；如果你想实现多继承，那么必须实用接口；
- 如果基本功能在不断变化，那么就需要使用抽象类，如果不断改变基本功能并且使用接口，那么就要改变所有实现了该接口的类。

# 10.  <span id="java_base_10">抽象类是否可以没有方法和属性？</span>

可以，抽象类可以没有方法和属性，但是含有抽象方法的类一定是抽象类。



# 11.  <span id="java_base_11">接口的意义</span>

规范、扩展、回调。

对一些类是否具有某个功能非常关心，但不关心功能的具体实现，那么就需要这些类实现一个接口。



# 12.  <span id="java_base_12">泛型中的extends和super的区别</span>

```
<? extends T>和<? super T>是泛型中的“通配符”和“边界”的概念。
<? extends T>：是指“上界通配符”，不能往里存，只能往外取。
<? super T>：是指“下界通配符”，不影响往里存，但往外取只能放在Object对象里。
```

为什么使用通配符？

在Java中，类和数组之间的对象关系是可以继承的。但是集合不存在这种关系。例如List<Animal>不是List<Dog>的父类。

为了建立两个集合之间的联系，就需要用到通配符。


**PECS原则：频繁往外读取内容，适合用上界Extends，经常往里插入的，适合用下界Super。**

参考自：[Java 通配符简析](https://wenzhiquan.github.io/2019/03/02/2019-03-02-java-wildcards/)



# 13.  <span id="java_base_13">父类的静态方法能否被子类重写？</span>

不能，子类继承父类后，非静态方法覆盖父类的方法，父类的静态方法被隐藏。



## 14. <span id="java_base_14"> 进程和线程的区别</span>

进程是资源分配的基本单位，线程是处理器调度的基本单位。

进程拥有独立的地址空间，线程没有独立的地址空间，同一个进程内多个线程共享其资源。

线程划分尺度更小，所以多线程程序并发性更高。

一个程序至少有一个进程，一个进程至少有一个线程。

线程是属于进程的，当进程退出时该进程所产生的线程都会被强制退出并清除。

线程占用的资源要少于进程所占用的资源。



# 15.  <span id="java_base_15">final、finally、finalize的区别</span>

final用于声明属性、方法和类，分别表示属性不可变，方法不可覆盖，类不可继承。

finally是异常处理语句结构的一部分，表示总是执行。

finalize是Object类的一个方法，在垃圾收集器执行的时候会回调被回收对象的finalize()方法，可以覆盖此方法提供垃圾收集时其他资源的回收，例如关闭文件。



# 16.  <span id="java_base_16">序列化的方式</span>

实现Serializable接口和实现Parcelable接口。



# 17.  <span id="java_base_17">Serializable 和 Parcelable 的区别</span>

Serializable 是 Java的序列化接口。特点是简单，直接实现该接口就行了，其他工作都被系统完成了，但是对于内存开销大，序列化和反序列化需要很多的 I/O 流操作。

Parcelable 是Android的序列化方式，主要用于在内存序列化上，采用该方法需要实现该接口并且手动实现序列化过程，相对复杂点。如果序列化到存储设备（文件等）或者网络传输，比较复杂，建议用Serializable 方式。

**最大的区别就是存储媒介的不同：**Serializable 使用IO读写存储在硬盘上，Parcelable 是直接在内存中读写，内存读写速度远大于IO读写，所以Parcelable 更加高效。Serializable在序列化过程中会产生大量的临时变量，会引起频繁的GC，效率低下。



# 18.  <span id="java_base_18">静态属性和静态方法是否可以被继承？是否可以被重写？以及原因。</span>

静态属性和静态方法可以被继承，但是没有被重写而是被隐藏。

静态方法和属性是属于类的，调用的时候直接通过 类名.方法名 完成，不需要继承机制就可以调用。如果子类里面定义了静态方法和属性，那么这时候父类的静态方法或属性称之为“隐藏”。至于是否继承一说，子类是有继承静态方法和属性，但是跟实例方法和属性不太一样，存在隐藏这种情况。

多态之所以能够实现依赖于继承、接口和重写、重载（继承和重写最为关键）。**有了继承和重写就可以实现父类的引用指向不同子类的对象。** 重写的功能是：重写后子类的优先级高于父类的优先级，但是隐藏是没有这个优先级之分的。

静态属性、静态方法和非静态的属性都可以被继承和隐藏而不能被重写，因此不能实现多态，不能实现父类的引用可以指向不同子类的对象。

非静态方法可以被继承和重写，因此可以实现多态。



# 19.  <span id="java_base_19">静态内部类的设计意图</span>

只是为了降低包的深度，方便类的使用，静态内部类适用于包含类当中，但又不依赖于外在的类，不用使用外在类的非静态属性和方法，只是为了方便管理类结构而定义。在创建静态内部类的时候，不需要外部类对象的引用。

非静态内部类有一个很大的优点：可以自由使用外部类中的变量和方法。

```java
static class Outer {
	class Inner {}
	static class StaticInner {}
}

Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
Outer.StaticInner inner0 = new Outer.StaticInner();
```

参考自：[https://www.zhihu.com/question/28197253](https://www.zhihu.com/question/28197253)



# 20.  <span id="java_base_20">成员内部类、静态内部类、方法内部类（局部内部类）和匿名内部类的理解，以及项目中的应用</span>

成员内部类：

	最普通的内部类，它的定义位于一个类的内部，这样看起来，成员内部类相当于外部类的一个成员，成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括private成员和静态成员）。不过需要注意的是，当成员内部类拥有和外部类同名的成员变量或者方法时，会发生隐藏现象，即默认情况下访问的是成员内部类的成员。如果要访问外部类的同名成员，形式如下：

```java
外部类.this.成员变量
外部类.this.成员方法
```

虽然成员内部类可以无条件的访问外部类的成员，而外部类想访问成员内部类的成员却不是那么随心所欲了。成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提必须存在一个外部类对象。创建成员内部类对象的一般方式如下：

```java
public class Test {
    public static void main(String[] args)  {
        //第一种方式：
        Outter outter = new Outter();
        Outter.Inner inner = outter.new Inner();  //必须通过Outter对象来创建
         
        //第二种方式：
        Outter.Inner inner1 = outter.getInnerInstance();
    }
}
 
class Outter {
    private Inner inner = null;
    public Outter() {
         
    }
     
    public Inner getInnerInstance() {
        if(inner == null)
            inner = new Inner();
        return inner;
    }
      
    class Inner {
        public Inner() {
             
        }
    }
}
```

内部类可以拥有private、protected、public以及包访问权限。如果成员内部类用private修饰，则只能在外部类的内部访问，如果用public修饰，则任何地方都能访问，如果用protected修饰，则只能在同一个包下或者继承外部类的情况下访问，如果是默认访问权限，则只能是同一个包下访问。这一点和外部类有一点不一样，外部类只能被public和包访问两种权限修饰。由于成员内部类看起来像外部类的一个成员，所以可以像类的成员一样拥有多种修饰权限。

静态内部类：

	静态内部类也是定义在另一个类里面的类，只不过在类的前面多了一个关键字static。静态内部类是不需要依赖外部类的，这点和类的静态成员属性有点相似，并且它不能使用外部类的非static成员变量或者方法。这点很好理解，因为在没有外部类的对象的情况下，可以创建静态内部类的对象，如果允许访问外部类的非static成员就会产生矛盾，因为外部类的非static成员必须依赖于具体的对象。

局部内部类：

	局部内部类是定义在一个方法或者一个作用域里面的的类，它和成员内部类的区别在于局部内部类的访问权限仅限于方法内或者该作用域内。注意，局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。

匿名内部类：

	匿名内部类不能有访问修饰符和static修饰符的，一般用于编写监听代码。匿名内部类是唯一一个没有构造器的类。正因为没有构造器，所以匿名内部类的适用范围非常有限，大部分匿名内部类用于接口回调。一般来说，匿名内部类用于继承其他类或者实现接口，并不需要增加额外的方法，只是对继承方法的实现或者是重写。

应用场景：

1. 最重要的一点，每个内部类都能独立的继承一个接口的实现，无论外部类是否已经继承了某个接口的实现，对于内部类都没有影响。内部类使得多继承的解决方案变得完整。

    2. 方便将存在一定逻辑关系的类组织在一起，又可以对外界隐蔽。
       3. 方便编写事件驱动程序
       4. 方便编写线程代码

总结：

	对于成员内部类，必须先产生外部类的实例化对象，才能产生内部类的实例化对象。而非静态内部类不用产生外部类的实例化对象即可产生内部类的实例化对象。

```java
创建静态内部类对象的一般形式：
外部类类名.内部类类名 xxx = new 外部类类名.内部类类名()
创建成员内部类对象的一般形式：
外部类类名.内部类类名 xxx = new 外部类对象名.new 内部类类名()
```

参考自：[http://www.cnblogs.com/latter/p/5665015.html](http://www.cnblogs.com/latter/p/5665015.html)



# 21.  <span id="java_base_21">谈谈对kotlin的理解</span>

Kotlin是一种静态类型的编程语言，可以运行在Java虚拟机上，并且也可以被编译成JavaScript代码。Kotlin旨在解决Java语言中存在的一些问题，例如空指针异常和过多的样板代码。

Kotlin具有简洁、安全、互操作性和可伸缩性等特点。它支持函数式编程风格，并且具有许多现代化语言的特性，如Lambda表达式、扩展函数、数据类、协程等等。此外，Kotlin还提供了很好的null安全支持，通过强制非空类型来减少空指针异常的发生。

Kotlin的代码比Java代码更加简洁明了，同时也保证了类型安全，这使得程序的可读性和可维护性都有了很大的提高。由于支持与Java无缝互操作，因此Kotlin与Java之间可以轻松进行混合编程，这也为正在使用Java的开发者们提供了更多的选择。

总之，Kotlin是一门功能强大的语言，它对Java的改进使得它更加适用于现代编程环境，同时也带给了开发者更多的灵活性和效率。


# 22.  <span id="java_base_22">闭包和局部内部类的区别</span>

所谓闭包，就是在函数中有另一个函数，这个内部函数可以作为参数，外部通过传递的方式，将函数传递进来。从而内部函数可以访问到外部函数的局部变量。



# 23.  <span id="java_base_23">String转换成Integer的方式以及原理</span>

String转int：

```java
int i = Integer.parseInt(s);
int i = Integer.valueOf(s).intValue();
```

int转String：

```java
String s = i + "";
String s = Integer.toString(i);
String s = String.valueOf(i);
```

源码：

```java
public static int parseInt(String s, int radix)  throws NumberFormatException  
    {  
        /* 
         * WARNING: This method may be invoked early during VM initialization 
         * before IntegerCache is initialized. Care must be taken to not use 
         * the valueOf method. 
         */  
    // 下面三个判断好理解，其中表示进制的 radix 要在（2~36）范围内  
        if (s == null) {  
            throw new NumberFormatException("null");  
        }  
  
        if (radix < Character.MIN_RADIX) {  
            throw new NumberFormatException("radix " + radix +  
                                            " less than Character.MIN_RADIX");  
        }  
  
        if (radix > Character.MAX_RADIX) {  
            throw new NumberFormatException("radix " + radix +  
                                            " greater than Character.MAX_RADIX");  
        }  
  
        int result = 0; // 表示结果， 在下面的计算中会一直是个负数，   
            // 假如说 我们的字符串是一个正数  "7" ，   
            // 那么在返回这个值之前result保存的是 -7，  
            // 这个可能是为了保持正数和负数在下面计算的一致性  
        boolean negative = false;  
        int i = 0, len = s.length();  
  
  
    //limit 默认初始化为 最大正整数的  负数 ，假如字符串表示的是正数，  
    //那么result(在返回之前一直是负数形式)就必须和这个最大正数的负数来比较，判断是否溢出  
  
        int limit = -Integer.MAX_VALUE;   
              
        int multmin;  
        int digit;  
  
        if (len > 0) {     // 首先是对第一个位置判断，是否含有正负号  
            char firstChar = s.charAt(0);  
            if (firstChar < '0') { // Possible leading "+" or "-"  
                if (firstChar == '-') {  
                    negative = true;  
              
            // 这里，在负号的情况下，判断溢出的值就变成了 整数的 最小负数了。  
                    limit = Integer.MIN_VALUE;  
  
                } else if (firstChar != '+')  
                    throw NumberFormatException.forInputString(s);  
  
                if (len == 1) // Cannot have lone "+" or "-"  
                    throw NumberFormatException.forInputString(s);  
                i++;  
            }  
  
            multmin = limit / radix;  
        // 这个是用来判断当前的 result 在接受下一个字符串位置的数字后会不会溢出。  
        //  原理很简单，为了方便，拿正数来说  
        //  （multmin result 在计算中都是负数），假如是10  
        // 进制,假设最大的10进制数是 21，那么multmin = 21/10 = 2,   
        //  如果我此时的 result 是 3 ，下一个字符c来了，result即将变成  
        // result = result * 10 + c;那么这个值是肯定大于 21 ，即溢出了，  
            //  这个溢出的值在 int里面是保存不了的，不可能先计算出  
        // result（此时的result已经不是溢出的那个值了） 后再去与最大值比较。    
        // 所以要通过先比较  result < multmin   （说明result * radix 后还比 limit 小）     
  
            while (i < len) {  
                // Accumulating negatively avoids surprises near MAX_VALUE  
                digit = Character.digit(s.charAt(i++),radix);  
                if (digit < 0) {  
                    throw NumberFormatException.forInputString(s);  
                }  
  
  
        // 这里就是上说的判断溢出，由于result统一用负值来计算，所以用了 小于 号  
        // 从正数的角度看就是   reslut > mulmin  下一步reslut * radix 肯定是 溢出了  
                if (result < multmin) {   
                    throw NumberFormatException.forInputString(s);  
                }  
  
  
  
                result *= radix;  
  
        // 这里也是判断溢出， 由于是负值来判断，相当于 （-result + digit）> - limit  
        // 但是不能用这种形式，如果这样去比较，那么得出的值是肯定判断不出溢出的。  
        // 所以用    result < limit + digit    很巧妙  
                if (result < limit + digit) {  
  
                    throw NumberFormatException.forInputString(s);  
                }  
                result -= digit; // 加上这个 digit 的值 （这里减就相当于加）  
            }  
        } else {  
            throw NumberFormatException.forInputString(s);  
        }  
        return negative ? result : -result; // 正数就返回  -result   
    } 
```

参考自：[http://blog.csdn.net/stu_wanghui/article/details/38564177](http://blog.csdn.net/stu_wanghui/article/details/38564177)



# 24.  <span id="java_base_24">面向对象思想</span>

### 设计原则 S.O.L.I.D

| 简写   | 翻译     |
| ---- | ------ |
| SRP  | 单一职责原则 |
| OCP  | 开放封闭原则 |
| LSP  | 里氏替换原则 |
| ISP  | 接口分离原则 |
| DIP  | 依赖倒置原则 |

**单一职责原则**

> 修改一个类的原因应该只有一个

换句话说就是让一个类只负责一件事，当这个类需要做过多的事情的时候，就需要分解这个类。

如果一个类承担的职责过多，就等于把这些职责耦合在了一起，一个职责的变化可能会削弱这个类完成其它职责的能力。

**开放封闭原则**

> 类应该对扩展开放，对修改关闭

扩展就是添加新功能的意思，因此该原则要求在添加新功能时不需要修改代码。

符合开闭原则最典型的设计模式是装饰者模式，它可以动态地将职责附加到对象上，而不用去修改类的代码。

**里氏替换原则**

> 子类对象必须能够替换掉所有父类对象

继承是一种 IS - A 关系，子类需要能够当成父类来使用，并且需要比父类更特殊。

如果不满足这个原则，那么各个子类的行为上就会有很大差异，增加继承体系的复杂度。

**接口分离原则**

> 不应该强迫客户依赖于它们不用的方法

因此使用多个专门的接口比使用单一的总接口更好。

**依赖倒置原则**

> 高层模块不应该依赖于低层模块，两者都应该依赖于抽象。
>
> 抽象不应该依赖于细节，细节应该依赖于抽象。

高层模块包含一个应用程序中重要的策略选择和业务模块，如果高层模块依赖于低层模块，那么低层模块的改动就会直接影响到高层模块，从而迫使高层模块也需要改动。

依赖于抽象意味着：

* 任何变量都不应该持有一个指向具体类的指针或者引用
* 任何类都不应该从具体类派生
* 任何方法都不应该覆写它的任何基类中的已经实现的方法

### 三大特性

**封装**

利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分离的独立实体。数据被保护在抽象数据类型的内部，尽可能的隐藏内部细节，只保留一些对外接口使之与外部发生联系。用户无需知道对象内部的细节，但可以通过对象对外提供的接口来访问该对象。

优点：

* 降低耦合：可以独立开发、测试、优化和修改等
* 减轻维护的负担：可以更容易的被程序猿理解，并且在调试的时候可以不影响其他模块
* 有效地调节性能：可以通过剖析确定哪些模块影响了系统的性能
* 提高软件的可重用性
* 降低了构建大型系统的分享：即使整个系统不可用，但是这些独立的模块却有可能是可用的

**继承**

继承实现了 IS - A 关系，继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。

**多态**

多态分为编译时多态和运行时多态。编译时多态主要指方法的重载，运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定。

运行时多态有三个条件：

* 继承
* 覆盖
* 向上转型

# 25. <span id="java_base_25">对象拷贝理解？深拷贝、浅拷贝的区别？</span>

首先要先明白为什么需要使用克隆呢？

克隆的对象可能包含一些已经修改过的属性，而 new 出来的对象的属性都还是初始化时候的值，所以当需要一个新的对象来保存当前对象的 "状态" 就需要克隆了。

那如何实现对象克隆呢？有两种办法：

1. 实现 Cloneable 接口并重写 Object 类中的 clone() 方法
2. 实现 Serialiable 接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆

**深拷贝和浅拷贝的区别是什么？**

浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
> 浅拷贝是指复制对象时，只复制对象的引用，而不复制对象本身。这意味着新对象和原对象将共享相同的内存地址。如果原对象发生变化，新对象也会受到影响。

深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

Java 默认的是浅拷贝，如果想实现深拷贝，就需要对象所包含的引用类型的成员变量也需要实现 Cloneable 接口，或者实现 Serialiable 接口。
> 深拷贝是指复制对象时，复制对象本身及其引用的对象。这意味着新对象和原对象不共享相同的内存地址。如果原对象发生变化，新对象不会受到影响。


## 26. <span id="java_base_26">Enumeration 和 Iterator 的区别？</span>

- 接口不同

  Enumeration 只能读取集合数据，而不能对数据进行修改；Iterator 除了读取集合的数据之外，也能对数据进行删除操作；Enumeration 已经被 Iterator 取代了，之所以没有被标记为 Deprecated，是因为在一些遗留类（Vector、Hashtable）中还在使用。

  ```java
  public interface Enumeration<E> {
      boolean hasMoreElements();
      E nextElement();
      default Iterator<E> asIterator() {
          return new Iterator<>() {
              @Override public boolean hasNext() {
                  return hasMoreElements();
              }
              @Override public E next() {
                  return nextElement();
              }
          };
      }
  }
  ```

  ```java
  public interface Iterator<E> {
      boolean hasNext();
      E next();
      default void remove() {
          throw new UnsupportedOperationException("remove");
      }
      default void forEachRemaining(Consumer<? super E> action) {
          Objects.requireNonNull(action);
          while (hasNext())
              action.accept(next());
      }
  }
  ```

- Iterator 支持 fail-fast 机制，而 Enumeration 不支持

  Enumeration 是 JDK 1.0 添加的接口。使用到它的函数包括 Vector、Hashtable 等类，Enumeration 存在的目的就是为它们提供遍历接口。

  Iterator 是 JDK 1.2 才添加的接口，它是为了 HashMap、ArrayList 等集合提供的遍历接口。Iterator 是支持 fail-fast 机制的。

  Fail-fast 机制是指 Java 集合 (Collection) 中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生 fail-fast 事件。例如：当某个线程 A 通过 Iterator 去遍历某集合的过程中，若该集合的内容被其他线程所改变了，那么线程 A 访问集合时，就会抛出 ConcurrentModificationException 异常，产生 fail-fast 事件。
