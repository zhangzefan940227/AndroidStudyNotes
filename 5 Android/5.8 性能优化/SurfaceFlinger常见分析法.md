# 图形界面显示

Android的GUI系统是基于OpenGL/EGL来实现的，一个界面能够显示需要两个动作：绘制和合成。

绘制：2D绘图利用hwui调用OpenGL,3D绘图直接调用OpenGL,skia用于绘制字体。
合成：软件合成是SF调用OpenGl合成输出到FrameBuffer,硬件合成是通过HWComposer直接进行Overlay。

# Surface

Surface 是Android系统中真正的画布，Activity上的所有UI都是在Surface 上完成绘制的，每一个Surface 对象都在SurfaceFlinger中有对应的图层（Layer），SurfaceFlinger 负责把这些Layer按需混合处理后输出到Frame Buffer中，再由Display设备（屏幕或显示器）把Frame Buffer里的数据呈现到屏幕上。

> 在ViewRootImpl.java里中涉及到Surface传递，Activity本身创建的Surface为空壳子，通过系统的SurfaceControl进行赋值，通过SurfaceControl是用来截图的就是这个原理。

# Android GUI系统简述
https://www.cnblogs.com/samchen2009/p/3364327.html

## Window,PhoneWindow,Activity

- **Activity** 是 Android 应用的四大组件之一 （Activity， Service,  Content Provider,  Broadcast Receiver), 也是唯一一个与用户直接交互的组件。

- Window 在 不同的地方有着不同的含义。
    - 在Activity里，Window 是一个抽象类，代表了一个矩形的不可见的容器，里面布局着若干个可视的区域（View). 每个Activity都会有一个Window类成员变量mWindow。
    - 而在WindowManagerService里，Window指的是WindowState对象，WindowState与一个ViewRootImpl里的mWindow对象相对应。所以说，WindowManagerService里管理的Window其实是Acitivity的ViewRoot。我们下面提到的Window，如果没有做特殊说明，均指的是WindowManagerService里的 ‘Window' 概念，即一个特定的显示区域。
    - 从用户角度来看，Android是个多窗口的操作系统，不同尺寸的窗口区域根据尺寸，位置，z-order及是否透明等参数 叠加起来一起并最终呈现给用户。这些窗口既可以是来自一个应用，也可以来自与多个应用，这些窗口既可以显示在一个平面，也可以是不同的平面。总而言之，窗 口是有层次的显示区域，每个窗口在底层最终体现为一个个的矩形Buffer, 这些Buffer经过计算合成为一个新的Buffer，最终交付Display系统进行显示。
    - 为了辅助最后的窗口管理，Android定义了一些不同的窗 口类型：
        - 应用程序窗口 (Application Window): 包括所有应用程序自己创建的窗口，以及在应用起来之前系统负责显示的窗口。
        - 子窗口(Sub Window)：比如应用自定义的对话框，或者输入法窗口，子窗口必须依附于某个应用窗口（设置相同的token)。
        - 系统窗口(System Window): 系统设计的，不依附于任何应用的窗口，比如说，状态栏(Status Bar), 导航栏(Navigation Bar), 壁纸(Wallpaper), 来电显示窗口(Phone)，锁屏窗口(KeyGuard), 信息提示窗口(Toast)， 音量调整窗口，鼠标光标等等。

- **PhoneWindow** 是Activity Window的扩展，是为手机或平板设备专门设计的一个窗口布局方案，就像大家在手机上看到的。

## View, DecorView, ViewGroup, ViewRoot
**View** 是一个矩形的可见区域。

**ViewGroup** 是一种特殊的View, 它可以包含其他View并以一定的方式进行布局。Android支持的布局有FrameLayout, LinearLayout, RelativeLayout 等。

**DecorView** 是FrameLayout的子类，FrameLayout 也叫单帧布局，是最简单的一种布局，所有的子View在垂直方向上按照先后顺序依次叠加，如果有重叠部分，后面的View将会把前面的View挡住。我们 经常看到的弹出框，把后面的窗口挡住一部分，就是用的FrameLayout布局。Android的窗口基本上用的都是FrameLayout布局, 所以DecorView也就是一个Activity Window的顶级View, 所有在窗口里显示的View都是它的子View.

**ViewRoot** 我们可以定义所有被addView()调用的View是ViewRoot, 因为接口将会生成一个ViewRootImpl 对象，并保存在WindowManagerGlobal的mRoots[] 数组里。一个程序可能有很多了ViewRoot（只要多次调用addView()), 在WindowManagerService端看来，就是多个Window。但在Activity的默认实现里，只有mDecorView 通过addView 添加到WindowManagerService里( 见如下代码)

