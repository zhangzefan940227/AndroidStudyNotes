# 背景

init经过前两个阶段后，已经建立了属性系统和SELinux系统，但是init进程还需要执行很多其他的操作，还要启动许多关键的系统服务，但是如果都是像属性系统和SELinux系统那样一行行代码去做，显得有点杂乱繁琐，而且不容易扩展，所以Android系统引入了init.rc。

init.rc是init进程启动的配置脚本，这个脚本是用一种叫Android Init Language(Android初始化语言)的语言写的，在7.0以前，init进程只解析根目录下的init.rc文件，但是随着版本的迭代，init.rc越来越臃肿，所以在7.0以后，init.rc一些业务被分拆到/system/etc/init，/vendor/etc/init，/odm/etc/init三个目录下。

# Android Init Language

- .rc 文件主要配置了两个东西，一个是action，一个是service。
    - trigger和command是对action的补充，options是对service的补充。
    - action加trigger以及一些command,组成一个Section；service加上一些option，也组成一个Section。.rc 文件就是由一个个的Section组成
- .rc 文件头部有一个import语法，表示这些引入的.rc文件也一并包含并解析。

## action 

action的语法格式如下：

```java
on <trigger> [&& <trigger>]*
    <command>
    <command>
    <command>
```

以on开头，trigger是判断条件，command是具体执行一些操作，当满足trigger条件时，执行这些command。

trigger可以是一个字符串，也可以是属性；条件可以有多个：

```java
on early //表示当trigger early或QueueEventTrigger("early")调用时触发
on property:sys.boot_from_charger_mode=1	//表示当sys.boot_from_charger_mode的值通过property_set设置为1时触发
on property:sys.sysctl.tcp_def_init_rwnd=*	// *表示任意值
    
//表示当zygote-start触发并且ro.crypto.state属性值为unencrypted时触发
on zygote-start && property:ro.crypto.state=unencrypted
```



## service

service的语法格式如下：

```java
service <name> <pathname> [ <argument> ]*
    <option>
    <option>
    ...
```

以service开头，name是指定这个服务的名称，pathname表示这个服务的执行文件路径，argument表示执行文件带的参数，option表示这个服务的一些配置。

以zygote服务为例：

```java
import /system/etc/init/hw/init.zygote64.rc

service zygote_secondary /system/bin/app_process32 -Xzygote /system/bin --zygote --socket-name=zygote_secondary --enable-lazy-preload
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote_secondary stream 660 root system
    socket usap_pool_secondary stream 660 root system
    onrestart restart zygote
    task_profiles ProcessCapacityHigh MaxPerformance
```

这个是配置是在 aosp/system/core/rootdir/init.zygote64_32.rc文件中的service，这个是zygote进程的启动配置。

zygote是进程名，system/bin/app_process32是可执行文件路径，后面的是执行参数。



# 参考链接

https://blog.csdn.net/marshal_zsx/article/details/80272760
https://www.cnblogs.com/lixuejian/p/15157634.html
