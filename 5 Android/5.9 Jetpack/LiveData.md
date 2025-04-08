# 介绍
LiveData是一种可观察的数据存储类。LiveData 具有生命周期感知能力，遵循其他应用组件（如 Activity、Fragment 或 Service）的生命周期。**这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的Observer，非活跃状态下的Observer不会受到通知。**

生命周期状态可以通过Lifecycle提供，包括DESTROYED、INITIALIZED、CREATED、STARTED、RESUMED，当且仅当生命周期处于STARTED、RESUMED时为活跃状态，其他状态是非活跃状态。

# LiveData核心方法
|方法名|	作用|
| -- | -- |
|observe(LifecycleOwner owner, Observer observer)|	注册和宿主生命周期关联的观察者， owner当前生命周期的宿主，当宿主销毁了observer能自动解除注册|
|observeForever(Observer observer)|	注册观察者，不会反注册，需自行维护，没有owner无法管理宿主生命周期|
|setValue(T value)|	发送数据，没有活跃的观察者时不分发，只能在主线程|
|postValue(T value)	|setValue一样，但是不受线程限制，内部也是通过handelr.post到主线程，最后还是通过setValue来分发的|
|onActive()|	当且仅当有一个活跃的观察者时会触发|
|onInactive()	|不存在活跃的观察者时会触发|

关键步骤：
1. 在Activity中注册观察者。
2. 在ViewModel中更新数据时，通过liveData发送数据（post/set)
