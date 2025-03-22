# 1 简介

### 1.1 标准广播

是一种完全异步执行的广播。

广播发出后，所有接收器几乎同一时间收到这条广播消息。

优点：效率高。缺点：无法被截断。

![](5%20Android/5.1%20%E5%9B%9B%E5%A4%A7%E7%BB%84%E4%BB%B6/img/e92cdaa93201137f2c5ecf88dbebcc91_MD5.png)

### 1.2 有序广播

是同步执行的广播。

广播发出后，同一时刻只有一个接收器能收到这条消息，当该广播接收器处理完消息之后，广播才会继续传递。因此，广播接收器有先后顺序。

优点：可以被截断。缺点：同步执行，效率低。

![](5%20Android/5.1%20%E5%9B%9B%E5%A4%A7%E7%BB%84%E4%BB%B6/img/4eba43fbfadc57b13657e6b5d16545fc_MD5.png)

# 2 动态注册广播

**创建一个广播接收器**

1. 新建一个类，继承 BroadcastReceiver 类。
2. 重写父类的 onReceive() 方法。

【实例】动态注册一个能够监听网络变化的程序

```java
public class MainActivity extends AppCompatActivity {
    private IntentFilter intentFilter;

    private NetworkChangeReceiver networkChangeReceiver;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        intentFilter = new IntentFilter();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver = new NetworkChangeReceiver();
        registerReceiver(networkChangeReceiver, intentFilter);
    }

    class NetworkChangeReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            ConnectivityManager connectivityManager = (ConnectivityManager)
                    getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
            if (networkInfo != null && networkInfo.isAvailable()) {
                Toast.makeText(context, "network is available",
                        Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(context, "network is unavailable",
                        Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```

思路：

1. 每当网络状态变化时，onReceive() 方法就会得到执行，这里只是简单的使用 Toast 提示了一段文本信息。
2. 创建的一个 IntentFilter 实例，并添加了一个值为 `android.net.conn.CONNECTIVITY_CHANGE` 的 action，因为网络状态发生变化时，发出的就是这条广播。
3. 创建一个 NetworkChangeReceiver 实例，然后调用 registerReceiver() 方法将 NetworkChangeReceiver 实例和 IntentFilter 实例都传进去。
4. 在 onReceive() 方法中，通过 getSystemService() 方法得到 ConnectivityManager 的实例，该类为系统服务类，用于管理网络连接。
5. 获取 NetworkInfo 的实例，并调用该实例的 isAvailable() 方法就可以判断出当前是否有网络。
6. 调用 getActiveNetworkInfo() 方法为获取网络状态，需要在 AndroidManifest.xml 中声明权限。

`<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />`

# 3 静态注册广播

动态注册广播在灵活性方面有很大的优势，但是它必须在程序启动之后才能接收到广播，因为注册的逻辑是写在 onCreate() 方法中的。

**同样创建一个广播接收器**

1. 新建一个类，继承 BroadcastReceiver 类。
2. 重写父类的 onReceive() 方法。
3. 静态广播接收器必须在 AndroidManifest.xml 文件中注册，在<receiver>标签中注册。
4. Android 系统在启动完成后会发出一条值为 android.intent.action.BOOT_COMPLETED 的广播，因此要在 <intent-filter>标签中添加相应的action
5. 监听系统开机广播需要声明权限，因此要加入 <uses-permission> 标签

```xml
<manifest >
  
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

	<application>
    <receiver
              android:name=".BootCompleteReceiver"
              android:enabled="true"
              android:exported="true">
    	<intent-filter>
      	<action android:name="android.intent.action.BOOT_COMPLETED" />
      </intent-filter>
    </receiver>
  </application>
</manifest>
```

# 4 发送自定义广播

**无序广播**

1. 定义一个广播接收器。
2. AndroManifest.xml 文件中对广播接收器添加<intent-filter> 标签，接收一条值为 com.example.broadcasttest.MY_BROADCAST 的 广播
3. 在主Activity类中，添加一个Button，通过点击该Button将广播发送出去。

```java
Button button = (Button) findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
	@Override
    public void onClick(View v) {
    	Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST");
        sendBroadcast(intent);	//无序广播
    }
});
```

**有序广播**

1. sendOrderedBroadcast() 方法中，第二个参数是一个与权限相关的字符串。

```java
Button button = (Button) findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
	@Override
    public void onClick(View v) {
    	Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST");
        sendOrderedBroadcast(intent, null);	//有序广播
    }
});
```

2. 有序广播在注册时设定广播接收器的顺序，在<intent-filte>标签中添加属性 `android:priority`，值越大优先级越高。

```xml
<intent-filter	android:priority="100">
  <anction android:name="com.example.broadcasttest.MY_BROADCAST" />
</intent-filter>
```

3. 如果在onReceive() 方法中调用了 abortBroadcast() 方法，就表示将这条广播截断了，后面的广播将无法收到广播。

# 5 使用本地广播

全局广播容易引起安全问题，因此Android 引入了本地广播机制

本地广播的使用和动态注册广播类似。主要是使用了一个 LocalBroadcastManager 来对广播进行管理。看一个示例代码：

```java
public class MainActivity extends AppCompatActivity {
    
    private IntentFilter intentFilter;
    private LocalReceiver localReceiver;
    private LocalBroadcastManager localBroadcastManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        localBroadcastManager = LocalBroadcastManager.getInstance(this);		//1. 获取实例。
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(
                        "com.example.broadcasttest.LOCAL_BROADCAST");		
                localBroadcastManager.sendBroadcast(intent);		//2. 发送本地广播
            }
        });
        intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.broadcasttest.LOCAL_BROADCAST");
        localReceiver = new LocalReceiver();
        localBroadcastManager.registerReceiver(localReceiver, intentFilter);		//3.注册本地广播监听器
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        localBroadcastManager.unregisterReceiver(localReceiver);
    }
}

class LocalReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "received local broadcast",
                Toast.LENGTH_SHORT).show();
    }
}
```

注意：

1. 本地广播只能给本应用发送，其他应用无法收到广播
2. 本地广播无法通过静态注册的方式来接收。（静态注册主要为了让程序在未启动的情况下收到广播，而发送本地广播时，应用肯定启动了。）