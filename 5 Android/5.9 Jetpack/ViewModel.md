d# 介绍
ViewModel的定义：ViewModel旨在以注重生命周期的方式存储和管理界面相关的数据。ViewModel本质上是视图（View）与数据（Model）之间的桥梁。

**ViewModel特点：**
- 生命周期比Activity长，数据可以在屏幕发生旋转、开关键盘等配置更改后继续留存。
- ViewModel中不能持有Activity引用，因此如果需要使用应用的Context（不可以使用Activity的Context，会导致内存泄露）则继承AndroidViewModel


