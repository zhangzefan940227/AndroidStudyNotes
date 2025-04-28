# 抓取systrace
adb wait-for-device
adb root
adb remount
for /F "tokens=1,2,3 delims=-:./\ " %%i in ("%DATE%") do SET F1=%%i%%j%%k
for /F "tokens=1,2,3 delims=-:./\ " %%i in ("%TIME%") do SET F2=%%i.%%j.%%k


set FILE_NAME=%F1%_%F2%.ptrace
set CURRENT_PATH=%cd%
adb shell "setprop persist.traced.enable 1"
adb shell "echo 0 > /d/tracing/tracing_on"
adb shell perfetto -t 20s -b 1gb -s 2gb --out /data/misc/perfetto-traces/%FILE_NAME%  gfx am input view sm wm res pm idle freq sched binder_driver bionic hal dalvik sync binder_driver -a com.moss.floatingtimer


adb pull /data/misc/perfetto-traces/%FILE_NAME% %CURRENT_PATH%/
adb shell  rm -rf  /data/misc/perfetto-traces/%FILE_NAME%

pause

# systrace分析
## 线程状态查看
### 绿色：运行中（Running）

​ 只有Running状态的线程才能在CPU中运行。同一时刻可以有多个线程处于Runnable状态，这些Runnable状态的线程的task_struct结构放入对应的CPU可执行队列中（同一时刻一个线程最多出现在一个CPU可执行队列中）。调度器 的任务就是从各个CPU的可执行队列中分别选择一个线程在该CPU上运行。

​ 通过查看Running状态的线程，根据其运行的时间和竞品的运行时间比较，分析快或者慢的原因。 频率不够、跑在了小核上、在Running和Runnable状态间频繁切换、在Running和Sleep状态之间频繁切换、是否跑在了不该跑的核上等。

### 蓝色：可运行（Runnable）

​ 线程可以运行，等待CPU调度。通过查看Runnable状态的持续时间可以判断CPU调度的速率。分析其调度快慢的原因，后台有太多任务、频率太低、被限制等。

### 白色：休眠中（Sleep）

​ 处于Sleep状态的线程是没有工作，可能是线程被阻塞。一般是在等事件驱动。

### 橘色 : 不可中断的睡眠态 （Uninterruptible Sleep - IO Block）

​ 线程在I / O上被阻塞或等待磁盘操作完成，一般底线都会标识出此时的 callsite ：wait_on_page_locked_killable。这个一般是标示 io 操作慢，如果有大量的橘色不可中断的睡眠态出现，那么一般是由于进入了低内存状态。

### 紫色 : 不可中断的睡眠态（Uninterruptible Sleep）

​ 线程在另一个内核操作（通常是内存管理）上被阻塞。一般是陷入了内核态，有些情况下是正常的，有些情况下是不正常的，需要按照具体的情况去分析。

