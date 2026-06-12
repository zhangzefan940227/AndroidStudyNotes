`ActivityThread`中实现`WindowManagerImpl`实例，并添加`View`
```java
// ActivityThread.java
@Override
public void handleResumeActivity(ActivityClientRecord r, boolean finalStateRequest,
        boolean isForward, boolean shouldSendCompatFakeFocus, String reason) {
    ... //省略
    if (r.window == null && !a.mFinished && willBeVisible) {
        r.window = r.activity.getWindow();
        View decor = r.window.getDecorView();
        decor.setVisibility(View.INVISIBLE);
        ViewManager wm = a.getWindowManager(); // 获取WindowManagerImpl对象
        WindowManager.LayoutParams l = r.window.getAttributes();
        a.mDecor = decor;
        l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
        l.softInputMode |= forwardBit;
        if (r.mPreserveWindow) {
            a.mWindowAdded = true;
            r.mPreserveWindow = false;
            // Normally the ViewRoot sets up callbacks with the Activity
            // in addView->ViewRootImpl#setView. If we are instead reusing
            // the decor view we have to notify the view root that the
            // callbacks may have changed.
            ViewRootImpl impl = decor.getViewRootImpl();
            if (impl != null) {
                impl.notifyChildRebuilt();
            }
        }
        if (a.mVisibleFromClient) {
            if (!a.mWindowAdded) {
                a.mWindowAdded = true;
                wm.addView(decor, l); // 添加DecorView
            } else {
                // The activity will get a callback for this {@link LayoutParams} change
                // earlier. However, at that time the decor will not be set (this is set
                // in this method), so no action will be taken. This call ensures the
                // callback occurs with the decor set.
                a.onWindowAttributesChanged(l);
            }
        }
    ... // 省略
```

`WindowManagerGlobal`中创建`ViewRootImpl`
```java
// WindowManagerGlobal.java
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow, int userId) {
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
    final Context context = view.getContext();
    if (parentWindow != null) {
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    } else {
        // If there's no parent, then hardware acceleration for this view is
        // set from the application's hardware acceleration setting.
        if (context != null
                && (context.getApplicationInfo().flags
                & ApplicationInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
            wparams.flags |= WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED;
        }
    }

    if (context != null && wparams.type > LAST_APPLICATION_WINDOW) {
        final TypedArray styles = context.obtainStyledAttributes(R.styleable.Window);
        if (PhoneWindow.isOptingOutEdgeToEdgeEnforcement(
                context.getApplicationInfo(), true /* local */, styles)) {
            wparams.privateFlags |= PRIVATE_FLAG_OPT_OUT_EDGE_TO_EDGE;
        }
        styles.recycle();
    }

    ViewRootImpl root;
    View panelParentView = null;

    synchronized (mLock) {
        ... // 省略
        if (windowlessSession == null) {
            root = new ViewRootImpl(view.getContext(), display);
        } else {
            root = new ViewRootImpl(view.getContext(), display,
                    windowlessSession, new WindowlessWindowLayout());
        }

        view.setLayoutParams(wparams);

        mViews.add(view); // 存储所有的DecorView
        mRoots.add(root); // 存储所有的ViewRootImpl
        mParams.add(wparams);

        // do this last because it fires off messages to start doing things
        try {
            root.setView(view, wparams, panelParentView, userId);
            mWindowViewsListenerGroup.accept(getWindowViews());
        } catch (RuntimeException e) {
            Log.e(TAG, "Couldn't add view: " + view, e);
            final int viewIndex = (index >= 0) ? index : (mViews.size() - 1);
            // BadTokenException or InvalidDisplayException, clean up.
            if (viewIndex >= 0) {
                removeViewLocked(viewIndex, true);
            }
            throw e;
        }
    }
}
```

在`ViewRootImpl`调用`setView`时会调用`WMS`的`addToDisplay`
```java
// ViewRootImpl.java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
    int userId) {
    synchronized (this) {
        if (mView == null) {
            ...
            
            // Schedule the first layout -before- adding to the window
            // manager, to make sure we do the relayout before receiving
            // any other events from the system.
            requestLayout();
            InputChannel inputChannel = null;
            if ((mWindowAttributes.inputFeatures
                    & WindowManager.LayoutParams.INPUT_FEATURE_NO_INPUT_CHANNEL) == 0) {
                inputChannel = new InputChannel(); // 初始化一个空的InputChannel
            }
            try {
                mOrigWindowType = mWindowAttributes.type;
                mAttachInfo.mRecomputeGlobalAttributes = true;
                collectViewAttributes();
                adjustLayoutParamsForCompatibility(mWindowAttributes,
                        mInsetsController.getAppearanceControlled(),
                        mInsetsController.isBehaviorControlled());
                controlInsetsForCompatibility(mWindowAttributes);

                Rect attachedFrame = new Rect();
                final float[] compatScale = { 1f };
                res = mWindowSession.addToDisplayAsUser(mWindow, mWindowAttributes, // mWindow为binder server端可以视为为应用进程的代表，这样WMS就可以通过Binder调用进入应用
                        getHostVisibility(), mDisplay.getDisplayId(), userId,
                        mInsetsController.getRequestedVisibleTypes(), inputChannel, mTempInsets,
                        mTempControls, attachedFrame, compatScale); // mWindowSession为binder的client端，最终调用的WMS，可以将其视为WMS的代表
                if (!attachedFrame.isValid()) {
                    attachedFrame = null;
                }
        }
    }
}
```

`IWindowSession`为匿名`Binder`，通过`WMS`的`openSession`创建
```java
// WindowManagerService.java
@Override
public IWindowSession openSession(IWindowSessionCallback callback) {
    return new Session(this, callback);
}
```

`Session`为`binder`的`Server`端，最终调用`WMS`的`addWindow`
```java
// frameworks\base\services\core\java\com\android\server\wm\Session.java
class Session extends IWindowSession.Stub implements IBinder.DeathRecipient {
    @Override
    public int addToDisplayAsUser(IWindow window, ..., InputChannel outInputChannel, ...) {
        return mService.addWindow(this, window, attrs, viewVisibility, displayId, userId,
                requestedVisibleTypes, outInputChannel, outInsetsState, outActiveControls,
                outAttachedFrame, outSizeCompatScale);
    }   
}
```

`WMS`的 `addView`
```java
    // WindowManagerService.java
    public int addWindow(Session session, IWindow client, LayoutParams attrs, int viewVisibility,
            int displayId, int requestUserId, @InsetsType int requestedVisibleTypes,
            InputChannel outInputChannel, InsetsState outInsetsState,
            InsetsSourceControl.Array outActiveControls, Rect outAttachedFrame,
            float[] outSizeCompatScale) {
        synchronized (mGlobalLock) {
            ...
            if (mWindowMap.containsKey(client.asBinder())) { // mWindowMap用来存储Window，key为IWindow对象，value为WindowState
                ProtoLog.w(WM_ERROR, "Window %s is already added", client);
                return WindowManagerGlobal.ADD_DUPLICATE_ADD;
            }
            
            ...
            if  (openInputChannels) {
                win.openInputChannel(outInputChannel);
            }
            
            
            mWindowMap.put(client.asBinder(), win); // 将WindowState添加到mWindowMap集合中
        }
    }
```

`WindowState`的`openInputChannel`
```java
    // WindowState.java
    void openInputChannel(@NonNull InputChannel outInputChannel) {
        if (mInputChannelToken != null) {
            throw new IllegalStateException("Window already has an input channel token.");
        }
        String name = getName();
        // 调用IMS的createInputChannel()
        InputChannel channel = mWmService.mInputManager.createInputChannel(name);
        mInputChannelToken = channel.getToken();
        mInputWindowHandle.setToken(mInputChannelToken);
        //WMS的mInputToWindowMap存放WindowState，key为InputChannel的token，value为WindowState
        mWmService.mInputToWindowMap.put(mInputChannelToken, this);
        //将返回的InputChannel复制到outInputChannel，此时InputChannel才由空壳变为实体
        channel.copyTo(outInputChannel);
        channel.dispose();
    }
```

`WMS`调用`IMS`的`createInputChannel`，通过JNI调用最后进入`com_android_server_input_InputManagerService.cp`p的`createInputChannel`
```cpp
base::Result<std::unique_ptr<InputChannel>> NativeInputManager::createInputChannel(
        const std::string& name) {
    ATRACE_CALL();
    return mInputManager->getDispatcher().createInputChannel(name);
}
```

`InputDispatcher.cpp`的`createInputChannel`，创建一对通信管道。Server端发送触摸事件，Client端接收事件。
```cpp
// InputDispatcher.cpp
Result<std::unique_ptr<InputChannel>> InputDispatcher::createInputChannel(const std::string& name) {
    LOG_IF(INFO, DEBUG_CHANNEL_CREATION) << "channel '" << name << "' ~ createInputChannel";

    //创建一对InputChannel，C/S结构
    std::unique_ptr<InputChannel> serverChannel;
    std::unique_ptr<InputChannel> clientChannel;
    
    //底层实际上是利用了Linux的socketpair系统调用，一次性创建出两个相互连接的Socket通道。函数执行成功后，这两个指针就会各自指向一个真正创建好的通道对象。
    status_t result = InputChannel::openInputChannelPair(name, serverChannel, clientChannel);

    if (result) {
        return base::Error(result) << "Failed to open input channel pair with name " << name;
    }

    { // acquire lock
        std::scoped_lock _l(mLock);
        const sp<IBinder>& token = serverChannel->getConnectionToken();
        
        // 组合一个回调函数，Server端数据到达之后触发回调函数handleReceiveCallback()
        std::function<int(int events)> callback = std::bind(&InputDispatcher::handleReceiveCallback,
                                                            this, std::placeholders::_1, token);
        
        // 将配置好的服务端通道、ID生成器和刚才绑定的回调函数，一起注册到系统的连接管理器中。
        mConnectionManager.createConnection(std::move(serverChannel), mIdGenerator, callback);
        // std::move(serverChannel)：C++ 的右值引用/移动语义。
        // 因为 serverChannel 是 unique_ptr（独占指针，不能被复制），
        // 所以必须使用 std::move 将该通道的所有权“移交/过户”给
        // mConnectionManager（连接管理器）。移交后，原变量 serverChannel 变为空。
        
    } // release lock

    // Wake the looper because some connections have changed.
    mLooper->wake();
    return clientChannel;
}
```

`InputTransport.cpp`的`openInputChannelPair`
```java
status_t InputChannel::openInputChannelPair(const std::string& name,
                                            std::unique_ptr<InputChannel>& outServerChannel,
                                            std::unique_ptr<InputChannel>& outClientChannel) {
    int sockets[2];
    if (socketpair(AF_UNIX, SOCK_SEQPACKET, 0, sockets)) {
        status_t result = -errno;
        ALOGE("channel '%s' ~ Could not create socket pair.  errno=%s(%d)", name.c_str(),
              strerror(errno), errno);
        outServerChannel.reset();
        outClientChannel.reset();
        return result;
    }

    //缓冲区的大小为32k
    int bufferSize = SOCKET_BUFFER_SIZE;
    setsockopt(sockets[0], SOL_SOCKET, SO_SNDBUF, &bufferSize, sizeof(bufferSize));
    setsockopt(sockets[0], SOL_SOCKET, SO_RCVBUF, &bufferSize, sizeof(bufferSize));
    setsockopt(sockets[1], SOL_SOCKET, SO_SNDBUF, &bufferSize, sizeof(bufferSize));
    setsockopt(sockets[1], SOL_SOCKET, SO_RCVBUF, &bufferSize, sizeof(bufferSize));

    sp<IBinder> token = sp<BBinder>::make();

    android::base::unique_fd serverFd(sockets[0]);
    //创建server端InputChannel
    outServerChannel = InputChannel::create(name, std::move(serverFd), token);

    android::base::unique_fd clientFd(sockets[1]);
    //创建client端InputChannel
    outClientChannel = InputChannel::create(name, std::move(clientFd), token);
    return OK;
}

std::unique_ptr<InputChannel> InputChannel::create(const std::string& name,
                                                   android::base::unique_fd fd, sp<IBinder> token) {
    //创建InputChannel的fd为高效无阻塞模式
    const int result = fcntl(fd, F_SETFL, O_NONBLOCK);
    if (result != 0) {
        LOG_ALWAYS_FATAL("channel '%s' ~ Could not make socket (%d) non-blocking: %s", name.c_str(),
                         fd.get(), strerror(errno));
        return nullptr;
    }
    // using 'new' to access a non-public constructor
    return std::unique_ptr<InputChannel>(new InputChannel(name, std::move(fd), token));
}
```


`IWindowSession`调用`addToDisplayAsUser`之后`InputChannel`对创建完毕，并且返回了客户端`InputChannel`，客户端`InputChannel`作为`WindowInputEventReceiver`的入参
```java
// ViewRootImpl.java
final class WindowInputEventReceiver extends InputEventReceiver {
    public WindowInputEventReceiver(InputChannel inputChannel, Looper looper) {
        super(inputChannel, looper);
    }
}
```

具体实现在`InputEventRecelver.java`中

```java
// InputEventRecelver.java
/**
 * Creates an input event receiver bound to the specified input channel.
 *
 * @param inputChannel The input channel.
 * @param looper The looper to use when invoking callbacks.
 */
public InputEventReceiver(InputChannel inputChannel, Looper looper) {
    // 从判空可以看出，必须初始化inputChannel和looper
    if (inputChannel == null) {
        throw new IllegalArgumentException("inputChannel must not be null");
    }
    if (looper == null) {
        throw new IllegalArgumentException("looper must not be null");
    }

    mInputChannel = inputChannel;
    mMessageQueue = looper.getQueue();
    mReceiverPtr = nativeInit(new WeakReference<InputEventReceiver>(this),
            mInputChannel, mMessageQueue);

    mCloseGuard.open("InputEventReceiver.dispose");
}
```

调用到`android_view_InputEventReceiver.cpp`
```cpp
// android_view_InputEventReceiver.cpp
static jlong nativeInit(JNIEnv* env, jclass clazz, jobject receiverWeak,
        jobject inputChannelObj, jobject messageQueueObj) {
    // 将java层的InputChannel转换成C++层的InputChannel
    std::shared_ptr<InputChannel> inputChannel =
            android_view_InputChannel_getInputChannel(env, inputChannelObj);
    if (inputChannel == nullptr) {
        jniThrowRuntimeException(env, "InputChannel is not initialized.");
        return 0;
    }

    // 将java层的MessageQueue转换成C++层的MessageQueue
    sp<MessageQueue> messageQueue = android_os_MessageQueue_getMessageQueue(env, messageQueueObj);
    if (messageQueue == nullptr) {
        jniThrowRuntimeException(env, "MessageQueue is not initialized.");
        return 0;
    }

    sp<NativeInputEventReceiver> receiver = new NativeInputEventReceiver(env,
            receiverWeak, inputChannel, messageQueue);
    status_t status = receiver->initialize();

    ...
    receiver->incStrong(gInputEventReceiverClassInfo.clazz); // retain a reference for the object
    return reinterpret_cast<jlong>(receiver.get());
}
```

`NativeInputEventRecelver`

```cpp
// android_view_InputEventReceiver.cpp
status_t NativeInputEventReceiver::initialize() {
    setFdEvents(ALOOPER_EVENT_INPUT);
    return OK;
}

void NativeInputEventReceiver::setFdEvents(int events) {
    if (mFdEvents != events) {
        mFdEvents = events;
        const int fd = mInputConsumer.getChannel()->getFd();
        if (events) {
            mMessageQueue->getLooper()->addFd(fd, 0, events, this, nullptr);
        } else {
            mMessageQueue->getLooper()->removeFd(fd);
        }
    }
}
```

如此便完成了将客户端socket的fd和服务端socket的fd添加到各自线程Looper，客户端socket的fd添加到应用主线程的Looper中，服务端socket的fd添加到。
