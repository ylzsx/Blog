## Handler

Android中，只有主线程才能操作UI，但是主线程不能进行耗时操作，否则会出现线程阻塞，产生ANR异常（Application not Responsing）。如果在子线程需要更新UI，一般是通过`Handler`发送消息，主线程接受消息并且进行相应的逻辑处理。

消息处理是通过 `Handler` 、`Looper` 以及 `MessageQueue`共同完成。 `Handler` 负责发送以及处理消息，`Looper` 创建消息队列并不断从队列中取出消息交给 `Handler`， `MessageQueue` 则用于保存消息。

### Handler实例

```java
public class MainActivity extends AppCompatActivity {
    private static final int MESSAGE_TEXT_VIEW = 0;
    
    private TextView mTextView;
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MESSAGE_TEXT_VIEW:
                    mTextView.setText("UI成功更新");
                default:
                    super.handleMessage(msg);
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        mTextView = (TextView) findViewById(R.id.text_view);

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                mHandler.obtainMessage(MESSAGE_TEXT_VIEW).sendToTarget();
            }
        }).start();
    }
}
```

### Handler构造方法部分源码：

```java
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

- 在构造方法中调用`mLooper = Looper.myLooper()`获得`mLooper`对象，若为空，则抛出异常，故**不能在未调用`Looper.prepare()`线程中创建`handler`**。
- 主线程在启动时会自动调用`Looper.prepare()`，故可以直接创建`handler`。
- 在得到`mLooper`中，获取其内部变量`mQueue`，得到`MessageQueue `消息对列对象，其内保存了`handler`中发送的消息。

### Looper

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

- 由上述代码可以看出`myLooper()`方法就是从本地线程对象`sThreadLocal`中获取到`Looper`对象，故`Looper`是线程独立的。

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

- `perpare()`方法保证了一个线程只会有一个Looper对象，同时在创建`Looper`对象时，内部会创建一个消息队列。

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;
    
    //无限循环遍历消息
    for (;;) {
        Message msg = queue.next(); // 得到下一条消息
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

		// 若成功获取消息(msg.target即为Handler对象),则进行发送消息
        msg.target.dispatchMessage(msg);

        msg.recycleUnchecked();
    }
}
```

- 由以上可知，`Looper`开启消息循环系统，不断从消息队列`MessageQueue`取出消息交由`Handler`。

### 消息发送和处理

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

- 在子线程发送消息时，是调用一系列的`sendMessage`、`sendMessageDelayed`以及`sendMessageAtTime`等方法，这些方法最终都会辗转调用`sendMessageAtTime(Message msg, long uptimeMillis)`。
- 该方法会调用`enqueueMessage`在消息队列中插入一条消息，同时会把`msg.target`设置为当前的`Handler`对象。
- 消息插入到消息队列中后，`Looper`负责从队列中取出，然后调用`Handler`的`dispatchMessage`方法。

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

- `dispatchMessage`方法首先会判断消息的`callback`方法是否为空，不为空则调用`handleCallback`方法处理，否则判断`Handler`的`mCallback`是否为空，不为空则调用它的`handleMessage`方法，如果仍为空，才会调用我们自己创建`Handler`时重写的`handleMessage`方法。
- 如果发送消息时调用`Handle`的`post(Runnable r)`方法，会把`Runnable`封装到消息对象的`callback`，然后调用`sendMessageDelayed`。此时在`dispatchMessage`中会调用`handleCallback`进行处理，即直接调用`Runnable`的`run()`方法。

```java
public final boolean post(Runnable r)
{
   return  sendMessageDelayed(getPostMessage(r), 0);
}
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

private static void handleCallback(Message message) {
    message.callback.run();
}
```

- 如果在创建`Handler`时，直接提供一个`Callback`对象，消息就交给了这个对象的`handleMessage`方法处理。`Callback`是`Handler`内部的一个接口。

```java
public interface Callback {
    public boolean handleMessage(Message msg);
}
```