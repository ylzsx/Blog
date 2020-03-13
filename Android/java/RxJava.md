## RxJava

RxJava能避免回调地狱。

登录前先初始化文件系统，初始化数据库系统。

```java
// 回调地狱
new Thread() {
    public void run() {
        // 初始化App目录
        initAppFiles(new InitSystemListener() {
            public void onSuccess(File file) {
                // 初始化目录成功后初始化数据库文件
                initSqliteData(new InitSqliteListener() {
                    public void onSuccess() {
                        getAppConfigFromWeb("http:www.192.168.1.102", new ...)
                    }
                })
            }
        })
    }
}
```

RxJava优点：链式调度、事件变换、线程切换。

异步回调、回调嵌套、跨线程消息机制。

使用的设计模式：观察者模式、构建者模式、责任链模式。

观察者模式：观察者、被观察者、抽象的被观察者、抽象的观察者。

### 泛型

extends 上界通配符，用于返回参数类型的限定，不能用于参数类型的限定。

super 下界通配符，用于参数类型的限定，不能用于返回参数的限定。

### 责任链模式

首先，需要一个抽象的处理者Handler，其中包含一个抽象的处理方法handle。真实处理者需要去实现handle方法。



## 博客中的RxJava

RxJava中基本概念：Observable(被观察者)、Observer(观察者)、subscribe(订阅)、event(事件)。`Observable` 和 `Observer` 通过 `subscribe()` 方法实现订阅关系，从而 `Observable` 可以在需要的时候发出事件来通知 `Observer`。

- `onNext()`：普通回调事件。

- `onCompleted()`：事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 `onNext()`发出时，需要触发 `onCompleted()` 方法作为标志。

- `onError()`: 事件队列异常。在事件处理过程中出异常时，`onError()` 会被触发，同时队列自动终止，不允许再有事件发出。

- 在一个正确运行的事件序列中, `onCompleted()` 和 `onError()` 有且只有一个，并且是事件序列中的最后一个。需要注意的是，`onCompleted()` 和 `onError()` 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

Subscriber是Observer的一个实现类，相比于Observer增加了两个方法：

- `onStart()`：在 subscribe刚开始，事件还未发送之前被调用，用来做一些准备工作。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行），`onStart()` 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 `doOnSubscribe()` 方法。
- `unsubscribe()`: 这是 `Subscriber` 所实现的另一个接口 `Subscription` 的方法，用于<font color='red'>取消订阅[若不取消订阅很容易出现内存泄露]</font>。在这个方法被调用后，`Subscriber`将不再接收事件。一般在这个方法调用前，可以使用 `isUnsubscribed()` 先判断一下状态。

`create()` 方法是 RxJava 最基本的创造事件序列的方法。基于这个方法， RxJava 还提供了一些方法用来快捷创建事件队列。

- `just(T...)`: 将传入的参数依次发送出来。
- `from(T[])` / `from(Iterable<? extends T>)` : 将传入的数组或 `Iterable`拆分成具体对象后，依次发送出来。     

调度器Scheduler -- 线程控制

- `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的 `Scheduler`。
- `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
- `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 `Scheduler`。行为模式和         `newThread()` 差不多，区别在于 `io()` 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 `io()` 比`newThread()` 更有效率。<font color='red'>不要把计算工作放在 `io()` 中</font>，可以避免创建不必要的线程。     
- `Schedulers.computation()`: 计算所使用的 `Scheduler`。这个计算指的是 CPU 密集型计算，即不会被 I/O等操作限制性能的操作，例如图形的计算。这个 `Scheduler` 使用的固定的线程池，大小为 CPU 核数。<font color='red'>不要把 I/O 操作放在 `computation()` 中</font>，否则 I/O 操作的等待时间会浪费 CPU。     
- Android 还有一个专用的 `AndroidSchedulers.mainThread()`，它指定的操作将在 Android 主线程运行。

`subscribeOn()`： 指定 `subscribe()` 所发生的线程，即事件的产生线程。

`observeOn()`：指定 `Subscriber` 所运行在的线程，即事件的消费线程。

- `observeOn()`指定的是执行时的当前 `Observable` 所对应的 `Subscriber`，即它的直接下级 `Subscriber` ，因此`observerOn()`可在每次想要切换线程的位置调用一次，故该方法可被<font color='red'>多次调用</font>。而`subscribeOn()`的位置放在哪里均可，但它<font color='red'>一般只调用一次</font>。

  > 观察者 Observer --> Subscriber    消费事件   多次	observeOn()
  > 被观察者 Observable   OnSubscribe 产生事件   一般一次     subscribeOn()

  <div>
      <img src="/resources/observeOn()原理图.png" style='border:0;display:inline'/>
      <img src="/resources/subscribeOn()原理图.png" style='border:0;display:inline'/>
  </div>

  > 注：当调用了多个`subscribeOn()`时，在整个通知过程中都会被第一个`subscribeOn()`截断，即只有第一个`subscribeOn()`起作用，见下图：
  >
  > ![/resources/多个subscribeOn()和observeOn()](/resources/多个subscribeOn()和observeOn().png)

`doOnSubscribe()`：在`subscribe()`调用后而且在事件发送前执行，区别于`onStart()`的是他可以指定线程，故其用来做需要指定线程的初始化工作。默认情况下， `doOnSubscribe()` 执行在 `subscribe()` 发生的线程；而如果在 `doOnSubscribe()` 之后有 `subscribeOn()` 的话，它将执行在离它最近的 `subscribeOn()`所指定的线程。

```java
Observable.create(onSubscribe)
    .subscribeOn(Schedulers.io()) 
    .doOnSubscribe(new Action0() {
        @Override 
        public void call() {
            progressBar.setVisibility(View.VISIBLE); // 需要在主线程执行 
        } 
    }).subscribeOn(AndroidSchedulers.mainThread()) // 指定主线程 
    .observeOn(AndroidSchedulers.mainThread()) 
    .subscribe(subscriber);
```

`doOnNext()`是观察者被通知之前(也就是回调之前)会调用的方法，即最终回调之前的一个回调方法，这个方法一般做的事件类似于观察者做的事情，只是自己不是最终的回调者。

 `map()` 是一对一的转化，而`flatMap()`是一对多的转化。

`flatMap()` 的原理：

- 使用传入的事件对象创建一个 `Observable` 对象；

- 并不发送这个 `Observable`, 而是将它激活，于是它开始发送事件；

- 每一个创建出来的 `Observable` 发送的事件，都被汇入同一个 `Observable`，而这个 `Observable` 负责将这些事件统一交给 `Subscriber` 的回调方法。这三个步骤，把事件拆成了两级，通过一组新创建的 `Observable` 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 `flatMap()` 所谓的 flat。

  ```java
  Student[] students = ...;
  Observable.from(students)
      .flatMap(new Func1<Student, Observable<Course>>() { 
          @Override 
          public Observable<Course> call(Student student) { 
              return Observable.from(student.getCourses()); 
          } 
      }).subscribe(new Subscriber<Couser>() {
      	@Override
          public void onNext(Course course) {
  			Log.d(tag, course.getName()); 
          }
  	});
  ```

  ```java
  // 通过flatMap做嵌套的网络请求
  networkClient.token() // 返回 Observable<String>，在订阅时请求 token，并在响应后发送 token 
  	.flatMap(new Func1<String, Observable<Messages>>() { 
          @Override 
          public Observable<Messages> call(String token) { 
              // 返回 Observable<Messages>，在订阅时请求消息列表，并在响应后发送请求到的消息列表 
              return networkClient.messages(); 
          } 
  	}).subscribe(new Action1<Messages>() { 
          @Override 
          public void call(Messages messages) { 
              // 处理显示消息列表 
              showMessages(messages); 
          }
  	});  
  ```

`throttleFirst()`: 在每次事件触发后的一定时间间隔内丢弃新的事件。常用作去抖动过滤，例如按钮的点击监听，防止在手机卡顿时用户点开重复的两个界面。

`compose()`：对Observable整体的变换：

## Rxjava 2.0

- Rxjava 2.0不再支持null值，如果传入null会抛出NullPointerException。

- Rxjava 2.0所有函数式接口设计为可抛出异常，解决编译异常需要转换问题。

- RxJava 1.0 中Observable不能很好支持背压，RxJava 2.0中将Oberservable彻底实现成不支持背压，而新增Flowable来支持非阻塞式的背压。

  在Rxjava中当一个快速发送消息的被观察者遇到一个处理消息缓慢的观察者，对于处理那些未处理消息的策略就称为背压策略。

### Observable

Observable数据流有两种类型：hot、cold。

- Cold observables

  只有当有订阅者订阅的时候，Cold Observable 才开始执行发射数据流的代码。并且每个订阅者订阅的时候都独立的执行一遍数据流代码。create/just/from创建的都是Cold observables。

- Hot observables

  Hot observable 不管有没有订阅者订阅，他们创建后就开发发射数据流。 比如鼠标事件，不管系统有没有订阅者监听鼠标事件，鼠标事件一直在发生，当有订阅者订阅后，从订阅后的事件开始发送给这个订阅者，之前的事件这个订阅者是接受不到的；如果订阅者取消订阅了，鼠标事件依然继续发射。

​	当一个cold observable是multicast（多路广播）的时候，为了应对背压，应当把cold observable转换成hot observable。cold observable 相当于响应式拉（即observer处理完了一个事件就从observable拉取下一个事件），hot observable通常不能很好的处理响应式拉模型，但它却是处理流量控制问题的不二候选人。

### 背压有关的操作符

- **Throttling**

  - sample()、throttleLast()：定期收集Observable发送的数据并发送最后一个数据item给观察者。

  - throttleFirst()：定期收集Observable发送的数据并发送第一个数据item给观察者。

    `Observable<Integer> burstyThrottled = bursty.throttleFirst(500, TimeUnit.MILLISECONDS);`

  - debounce()、throttleWithTimeout()：发送在指定持续时间段内未被其他item跟随的那个item。

- **Buffers and windows**

  可以使用`buffer()`或	`window()`运算符收集过度生成的Observable，并将他们作为一个集合发送给缓慢的消费者，消费者可以决定处理集合中的某个或者某些项。

  - buffer()：可以以固定的时间间隔定期关闭并一次性发送Observable缓冲区。

    可以通过`debounce()`操作符向缓存区发出缓存关闭信号，在突发期间将数据收集到缓冲区中，并在每个突发结束时将它们发出。

    ![buffer()示意图](/resources/buffer()示意图.png)

  - window()：可以以一定间隔定期发送Observable信息。

    `Observable<Observable<Integer>> burstyWindowed = bursty.window(500, TimeUnit.MILLISECONDS);`

    ![window()示意图](/resources/window()示意图.png)

    也可以在每次发送时生成一个新的窗口去发送特定数量的数据。

    `Observable<Observable<Integer>> burstyWindowed = bursty.window(5);`

### 建立"响应式拉动"的背压

当Subscriber订阅observable的时候可以在`onStart()`通过调用`subscribe.request(n)`，n是在下一次`request()`前发送的最大项数。当在`onNext()`方法中处理完上一次的数据项后，可以重新调用`request()`方法要求发送数据项。

```java
someObservable.subscribe(new Subscriber<t>() {
    @Override
    public void onStart() {
      request(1);
    }

    @Override
    public void onCompleted() {
      // gracefully handle sequence-complete
    }

    @Override
    public void onError(Throwable e) {
      // gracefully handle error
    }

    @Override
    public void onNext(t n) {
      // do something with the emitted item "n"
      // request another item:
      request(1);
    }
});
```

可以通过`request(Long.MAX_VALUE)`禁用"响应式拉动"背压，此时Observable会以自己的速度发送items。request(0)是一个合法的调用,但不会奏效。请求值小于零的请求会导致抛出异常。

### 解决背压（响应式拉动、堆栈阻塞式onBackpressureBlock）

当有两个发送频率不同的Observable，并且其中一个Observable不支持"响应式拉动"背压，那么可以使用一下操作符：

- **onBackpressureBuffer()**：为所有从源observable发送出来的数据项维护一个缓存区，根据他们生成的request发送给下层。

  ![onBackpressureBuffer()示意图](/resources/onBackpressureBuffer()示意图.png)

- **onBackpressureDrop()**：终止发送来自源observable的事件，除非来自下层流的subscriber即将调用request(n)方法的时候，此时才会发送足够的数据项给以满足requst。

  ![onBackpressureDrop()示意图](/resources/onBackpressureDrop()示意图.png)

- **onBackpressureBlock()**<font color='red'> (实验性的,   不支持RxJava 1.0)</font>

  ![onBackpressureBlock()示意图](/resources/onBackpressureBlock()示意图.png)

  阻塞源Observable正在操作的线程的线程直到某个Subscriber发出请求，然后只要有即将发出的请求就结束阻塞。

  > 当在不支持背压同时未使用以上操作符的Observable上尝试应用"响应式拉动"背压，将会在`onError()`中抛出`MissBackpresssureException`异常。

## Rxjava2.0 使用

- rxjava 2.0中取消了观察者`Subscriber`，取而代之的是 `ObservableEmitter`，同时在创建观察者时使用`Observer`，回调方法增加了一个`onSubscribe(Disposable d)`。

- 当调用Disposable对象的dispose() 方法时, 它就会将事件传送管道切断, 从而导致下游收不到事件，但上游会继续发送剩余的事件。

  ```java
  public void rxjava2() {
      // 1. 初始化Observable
      Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
          @Override
          public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
              emitter.onNext(1);
              emitter.onNext(2);
              emitter.onComplete();
          }
      });
      // 2. 创建一个观察者
      Observer<Integer> observer = new Observer<Integer>() {
          @Override
          public void onSubscribe(Disposable d) {}
  
          @Override
          public void onNext(Integer integer) {}
  
          @Override
          public void onError(Throwable e) {}
  
          @Override
          public void onComplete() {}
      };
      // 3. 建立订阅关系
      observable.subscribe(observer);
  
  
  
      // 简单订阅方式，可以使用lambda表达式
      Disposable disposable = Observable.fromArray(1).subscribe(new Consumer<Integer>() {
          @Override
          public void accept(Integer integer) throws Exception {
              // 相当于onNext
          }
      }, new Consumer<Throwable>() {
          @Override
          public void accept(Throwable throwable) throws Exception {
              // 相当于onError
          }
      }, new Action() {
          @Override
          public void run() throws Exception {
              // 相当于onComplete
          }
      }, new Consumer<Disposable>() {
          @Override
          public void accept(Disposable disposable) throws Exception {
              // 相当于onSubscribe
          }
      });
  }
  ```

- 取消了Fun/Action接口，改之为类似java8的函数式接口。

- Flowable并不是订阅就开始发送数据，而是需要等到执行`Subscription.request()`才开始发送数据。使用简化subscribe订阅方法会默认指定`Long.MAX_VALUE`。手动指定时通过在`onSubscribe()`指定。
- 新增`Processor`，继承自`Flowable`，支持背压控制，而`Subject`不支持背压控制。

- Disposable类

  - dispose():主动解除订阅。
  - isDisposed():查询是否解除订阅，true代表已经解除订阅。

- CompositeDisposable类：一个disposable的容器，可以容纳多个disposable，添加和去除的复杂度为O(1)。如果这个CompositeDisposable容器已经是处于dispose的状态，那么所有加进来的disposable都会被自动切断。

  可以快速解除所有添加的Disposable类，每当我们得到一个Disposable时就调用CompositeDisposable.add()将它添加到容器中, 在退出的时候, 调用CompositeDisposable.clear() 即可快速解除。