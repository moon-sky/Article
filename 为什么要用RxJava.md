##关于为什么要用RxJava
>原文链接[https://guides.codepath.com/android/RxJava#handling-configuration-changes](https://guides.codepath.com/android/RxJava#handling-configuration-changes)

>如果有侵权问题，请告知！
###概述
RxJava官方描述为“使用可观察序列构建异步和基于item的程序的库”。但这究竟意味着什么呢？让我们来一起聊一聊这个库。

编写健壮的Android应用程序的挑战之一是不断变化的输入的动态特性。在传统的命令式编程模型中，必须在变量上显式设置值以便更新它们。如果一个相关值发生更改，则在不添加另一行代码的情况下不会更新该值。请参考以下示例
```java
// init variables
int i, j, k; 

// Init inputs
i = 1;
j = 2;

// Set output value
k = i + j;

// Update a dependent value
j = 4;
k = ?  // What should k be?
```

诸如k之类的状态变量本质上可能无法反映其输入的当前值。传统的异步编程方法倾向于依靠回调来更新这些更改，但这种方式可能会导致称为[回调地狱](http://callbackhell.com/)的问题。Reactive编程（参见[此处](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)的介绍）通过提供描述输出以反映其变化的输入的框架来解决这些问题。RxJava是.NET的Reactive Extensions库的一个应用，使得Android应用程序能够以这种风格进行构建。查看Kaushik对[RxJava的视频](https://www.youtube.com/watch?v=k3D0cWyNno4)介绍，它提供了对库的可靠概述。有关教程，请参阅[this tutorial by Chris Arriola](https://github.com/arriolac/GitHubRxJava/wiki/Tutorial)的视频。
###优点
下面列出一些在Android平台使用RxJava的一些优点

*  **简化异步操作链作用** 如果您需要执行依赖于另一个API调用的API调用，您可能最终会在第一个调用的回调中实现此调用。RxJava提供了一种避免创建回调层以解决此问题的方法。出于这个原因，RxJava在2014年在Netflix中变得流行，以抽象出执行依赖API调用的复杂性。

* **展示了一种更明确的方式来声明并发操作应该如何操作**。虽然默认情况下RxJava是单线程的，但RxJava有助于您更明确地定义应该为后台和回调任务使用哪种类型的线程模型。由于Android仅允许主线程上的UI更新，因此使用RxJava有助于使代码更清楚地了解将更新视图所执行的操作。
* **暴露错误更快。**AsyncTask的一个问题是，在使用onPostExecute（）方法更新UI线程时，后台线程上发生的错误很难传递。此外，如[本博客文章](https://blog.danlew.net/2014/06/21/the-hidden-pitfalls-of-asynctask/)中所述，可以同运行多少个AsyncTasks存在限制。RxJava提供了一种方法来暴露这些错误。
* **有助于减少对可能引入bug的状态变量的需求。**使用RxJava时需要考虑的一个思维方式就是描述数据如何在系统中流动。用户触发的点击item，网络请求，来自外部源的数据更新都可以全部描述为异步数据流。RxJava的强大功能在于它可以转换，过滤或使用这些流来创建新的数据流，只需几行代码，同时还可以最大限度地减少存储状态变量的需求。

###搭建
设置你的build.gradle
```groovy
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
// Because RxAndroid releases are few and far between, it is recommended you also
// explicitly depend on RxJava's latest version for bug fixes and new features.
// (see https://github.com/ReactiveX/RxJava/releases for latest 2.x.x version)
implementation 'io.reactivex.rxjava2:rxjava:2.x.x'
```
如果您打算使用RxJava，强烈建议您设置[lambda表达式](https://guides.codepath.com/android/Lambda-Expressions)以减少通常需要的详细语法。该指南包含有关如何更新现有Android Studio项目以使用它的说明。
###被观察者 and 观察者

Reactive代码的基本构建块是Observables和Observers。Observable发出item;Observers处理这些item。Observable可以发出任意数量的item（包括无item），然后最终因为成功完成或由于错误而终止。

Observable可以拥有任意数量的观察者。对于每个相关的Observer，Observable为每个item调用Observer.onNext（），后跟Observer.onComplete（）或Observer.onError（）。请记住，需要至少有一个订阅者，Observable才会开始发送item。
####定义被观察者
让我们以最基本的例子来理解这是如何构建的。

首先，让我们定义一个Observable，它是一个可以发出任意数量要处理的item的对象：
```java
// Observables emit any number of items to be processed
// The type of the item to be processed needs to be specified as a "generic type"
// In this case, the item type is `String`
Observable<String> myObservable = Observable.create(
    new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> sub) {
            // "Emit" any data to the subscriber
            sub.onNext("a");
            sub.onNext("b");
            sub.onNext("c");
            // Trigger the completion of the event
            sub.onCompleted();
        }
    }
);
```
被观察者依次发送出 “a” “b” “ c” 然后完成
####定义观察者Observers
现在我们来定一个能够消化这些数据的监听者
```java
Observer<String> mySubscriber = new Observer<String>() {
    // Triggered for each emitted value
    @Override
    public void onNext(String s) { System.out.println("onNext: " + s); }

    // Triggered once the observable is complete
    @Override
    public void onCompleted() { System.out.println("done!"); }

    // Triggered if there is any errors during the event
    @Override
    public void onError(Throwable e) { }
};
```
####订阅被观察者（Observables）
要为这些item实现观察者，必须定义以下接口：
```java
public interface Observer<T> {
    void onNext(T t); // called for each "emitted" item
    void onCompleted(); // will not be called if onError() is called
    void onError(Throwable e); // called if there's an error
}
```
请注意，Observer是泛型类型。它必须表示Observable将发出的item的类型。对于订阅者开始监听将生成字符串类型的observable，它必须订阅Observable：
```java
Observable.just("a", "b", "c").subscribe(new Observer<String>() {
    // Triggered for each emitted value
    // Invoked with "a", then "b", then "c"
    @Override
    public void onNext(String s) { System.out.println("onNext: " + s); }

    // Triggered once the observable is complete
    @Override
    public void onCompleted() { System.out.println("done!"); }

    // Triggered if there is any errors during the event
    @Override
    public void onError(Throwable e) { }
});
```
上面的这个例子只是打印每个参数（“a”，“b”，“c”）然后退出，因为每个项都是通过调用onNext来调用的。调用所有项后，将调用onCompleted方法。

这可能看起来很刻意，在这个例子中并不是特别有用，但是作为展示建立在这些基础之上时的效果，我们开始看到RxJava的力量。

Observer可以监听Observable，以便响应发出的数据：
```java
// Attaches the subscriber above to the observable object
myObservable.subscribe(mySubscriber);
// Outputs:
// onNext: "a"
// onNext: "b"
// onNext: "c"
// done!
```
此示例将简单地打印每个发出的item然后退出，因为通过调用onNext处理每个item。一旦处理完了所有item，就会调用Observer的onCompleted方法。
####创建被监听者（observables）的其他方式
如上所示，Observer监视Observable发出的结果。当这些事件发生时，订阅者（observer）响应这些事件。可以从任何类型的输入创建Observable。

几乎任何被认为是异步数据流的东西。例如，Observable可以是一组可以迭代的字符串：
```java
// `just` generates an observable object that emits each letter and then completes the stream
Observable.just("a", "b", "c");
```
也可以使用已存在的数列来创建Observable
```java
ArrayList<String> items = new ArrayList();
items.add("red");
items.add("orange");
items.add("yellow");

Observable.from(items);
```
如果您尝试将内部代码转换为Observables，请考虑使用Observable.defer（），尤其是对于可能导致代码阻塞线程的代码：

```java
private Object slowBlockingMethod() { ... }

public Observable<Object> newMethod() {
    return Observable.just(slowBlockingMethod()); // slow blocking method will be invoked when created
}
```
相反，我们应该使用defer（）来创建observable：
```java
public Observable<Object> newMethod() {
    return Observable.defer(() -> Observable.just(slowBlockingMethod()));
}
```
通过查看该[博客](http://blog.danlew.net/2014/10/08/grokking-rxjava-part-4/#oldslowcode)来了解更多 细节。如果想要知道更多创建Observable的方式，请看[这里]（https://github.com/ReactiveX/RxJava/wiki/Creating-Observables）
####调度器（Schedulers）
默认情况下，RxJava是同步的，但可以使用调度程序异步定义工作。例如，我们可以定义网络调用应该在后台线程上完成，但回调应该在主UI线程上完成。
```java
Observable.from(Arrays.asList(1,2,3,4,5))
    .subscribeOn(Schedulers.newThread())
    .observeOn(AndroidSchedulers.mainThread()))
    .subscribe(new Subscriber<String>() {
        @Override
        public void onCompleted() {
            //called on completion
        }
        
        @Override
        public void onError(final Throwable e) {
            //called when error occurs
        }
        
        @Override
        public void onNext(final String s) {
            Log.d("emit", s);
        }
    });
```
以下是上面的示例代码中需要注意的事项：

* subscribeOn(Schedulers.newThread()): 这将使Observable在新线程中完成后台工作
* .observeOn（AndroidSchedulers.mainThread（）））：这使得订阅者在Android的主UI线程上执行其结果。这非常重要，尤其是在需要根据结果更改UI组件时。
* .subscribe（）：为Observable订阅一个Observer。Observers onNext方法在每个发出的项目上调用，如果成功运行则接着调用onCompletion，如果发生错误则返回onError。

使用调度器依赖于通过有界或无界线程池对工作进行排队。以下是RxJava附带的一些选项。有关所有可能的选项，请参阅此[链接](http://reactivex.io/RxJava/javadoc/rx/schedulers/Schedulers.html)

|Name|描述
|--|--|
Schedulers.computation()|固定的线程数 (= to # CPU's)
Schedulers.immediate()  |当前线程
Schedulers.io() |由可以根据需要扩展的线程池支持
Schedulers.newThread()  |新的独立线程
Schedulers.trampoline() |调度当前线程的工作但放在队列中

然后，可以使用这些调度器来控制可观察或订阅者使用subscribeOn（）和observeOn（）操作的线程。

调度器与RxJava开箱即用。RxAndroid库附带了AndroidSchedulers.mainThread（），这是访问Android上主UI线程的便捷方式。
####用Observable替换AsyncTask
我们可以使用Observable替换任何使用RxJava调用的AsyncTask调用。类似于AsyncTask在后台执行任务然后在UI上的主线程上调用onPostExecute（），RxJava可以通过定义使用subscribeOn（）执行任务的线程来完成同样的功能，然后使用observeOn（）定义回调的线程：
```java
// This constructs an `Observable` to download the image
public Observable<Bitmap> getImageNetworkCall() {
    // Insert network call here!
}

// Construct the observable and use `subscribeOn` and `observeOn`
// This controls which threads are used for processing and observing
Subscription subscription = getImageNetworkCall()
    // Specify the `Scheduler` on which an Observable will operate
    .subscribeOn(Schedulers.io()) 
    // Specify the `Scheduler` on which a subscriber will observe this `Observable`
    .observeOn(AndroidSchedulers.mainThread()) 
    .subscribe(new Subscriber<Bitmap>() {

        // This replaces `onPostExecute(Bitmap bitmap)`
        @Override
        public void onNext(Bitmap bitmap) {
             // Handle result of network request
        }

        @Override
        public void onCompleted() {
             // Update user interface if needed
        }

        @Override
        public void onError() {
             // Update user interface to handle error
        }

});
```
使用Observable，Subscriber和线程调度的这种组合，我们可以看到RxJava的强大功能。但是还有很多事情可以做。
####在Retrofit中应用Rxjava
RxJava可与Retrofit一起使用，以提供链接多个网络请求的功能。如果其中一个网络请求失败，它还提供了一个呈现错误的抽象概念。以下部分介绍在Retrofit如何使用RxJava。

Retrofit库只是将同步网络调用包装为Observable类型，以便与RxJava一起使用。将接口声明为Observable会自动执行此操作。
```java
public interface MyApiEndpointInterface {
  @GET("/users/{username}")
  Observable<User> getUser(@Path("username") String username);
}
```
我们可以通过实例化一个接口来获得一个observable类型对象。

```java
MyApiEndpointInterface apiService =
    retrofit.create(MyApiEndpointInterface.class);

// Get the observable User object
Observable<User> call = apiService.getUser(username);
// To define where the work is done, we can use `observeOn()` with Retrofit
// This means the result is handed to the subscriber on the main thread
call.subscribeOn(Schedulers.io())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new Subscriber<User>() {
    @Override
    public void onNext(User user) {
       // Called once the `User` object is available
    }

    @Override
    public void onCompleted() {
      // Nothing to do here
    }

    @Override
    public void onError(Throwable e) {
      // cast to retrofit.HttpException to get the response code
      if (e instanceof HttpException) {
         HttpException response;
         int code = response.code();
      }
    }
  });
```
RxAndroid提供了一个AndroidSchedulers.mainThread（）模式，允许在android UI线程处理相关事务。

####热监听与冷监听
默认情况下，Observables初始化为在第一个订阅者出现后开始执行。Retrofit默认情况下，会以这种方式运行，这被称为冷监听。您可以查看[Retrofit源代码](https://github.com/square/retrofit/blob/master/retrofit-adapters/rxjava/src/main/java/retrofit2/adapter/rxjava/RxJavaCallAdapterFactory.java#L92)，以查看是否在第一个订阅后发出了网络请求。
#####冷监听转为热监听
如果您希望更改它以便在执行请求之前添加多个订阅者（也称为转换为热可观察对象），则需要将Observable转换为ConnectableObservable。要启动网络请求，您需要在observable上调用connect（）：
```java
Observable<User> call = apiService.getUser(username);

// convert Observable to ConnectedObservable, get a reference so we can call connect() later
ConnectableObservable<User> connectedObservable = call.publish();

// define 1st observer
Observer<User> observer1 = new Observer<User>() {
   @Override
   public void onCompleted() {

   }

   @Override
   public void onError(Throwable e) {

   }

   @Override
   public void onNext(User user) {
      // do work here
   }
};        

// define 2nd observer here
Observer<User> observer2 = new Observer<User>() { 
}

// observer is subscribing
connectedObservable.observeOn(AndroidSchedulers.mainThread()).subscribe(observer1);
connectedObservable.observeOn(AndroidSchedulers.mainThread()).subscribe(observer2);

// initiate the network request
connectedObservable.connect();
```
####如何做到冷监听
您还可以使用autoConnect（）将热的可观察对象变回冷可观察对象。您可以使用此方法使下一个订阅者在下一次订阅时触发网络请求，而不需要调用显式connect（）并传递ConnectedObservable类型：
```java
// back to cold observable
Observable<User> observable = connectedObservable.autoConnect();  

// define 3rd observer here
Observer<User> observer3 = new Observer<User>() { 
}

observable.subscribe(observer3);
```
###处理配置更改
如果由于屏幕方向的改变而导致Activity被销毁，Observable将在onCreate（）中再次触发，这将导致重复工作，因为将再次进行相同的API调用，并且最初进行的所有进度都将丢失。要解决此问题，您需要首先使用单例或[能够保存信息的Fragment](https://guides.codepath.com/android/Handling-Configuration-Changes#retaining-fragments)存储对Observable的引用，并使用cache（）运算符：
```java
Observable<User> call = apiService.getUser(username).cache();
```
未来的观察者将在订阅后立即从Observables获得先前发出的item
```java
Subscription subscription = observable.subscribe(observer1);
```
如果您希望对缓冲区大小或将发出缓存事件的时间跨度进行更细粒度的控制，则应使用replay（）运算符：
```java
ConnectableObservable<User> call = apiService.getUser(username).replay(10, TimeUnit.MINUTES);
```
另请注意，replay（）将observable转换为hot observable，在调用connect（）之前不会发出事件

###避免内存泄漏
能够在不同线程上调度可观察量并使它们作为长时间运行的任务运行的灵活性可以很容易地导致内存泄漏。其中一个原因是观察者通常使用匿名类创建。在Java中，创建匿名类需要内部类将实例保留到包含的类，如本[Stack Overflow文章](http://stackoverflow.com/questions/5054360/do-anonymous-classes-always-maintain-a-reference-to-their-enclosing-instance)中所述。

因此，在Activity或Fragment中创建的观察者可以保存一个引用，如果observable仍在运行，则该引用将无法进行垃圾回收。针对以上问题，有几种不同的处理方式。这些方法都尝试通过管理观察者订阅observable的申请，并在发生生命周期事件时取消它们。
####Composite Subscription
最简单的处理处理方法是在你的Activity或者Fragment创建一个Composite Subscription实例.
```java
public class MainActivity extends AppCompatActivity {

    private CompositeSubscription subscriptions;

    protected void onCreate(Bundle savedInstanceState) {
       subscriptions = new CompositeSubscription();
    }
}
```
我们可以通过CompositeSubsciption来追踪通过add（）方法添加的申请
```java
subscriptions.add(connectedObservable.observeOn(AndroidSchedulers.mainThread()).subscribe(observer1))
subscriptions.add(connectedObservable.observeOn(AndroidSchedulers.mainThread()).subscribe(observer1));
```
应该在Activity或者Fragment被挂起的时候即回调onPause（）方法的时候，取消这些申请
```java
@Override
public void onPause() {
   super.onPause();

   if (subscriptions != null) {
      subscriptions.clear();
   }
}
```
一旦调用了clear()方法，CompositeSubscription可以被重复使用。
###RxBinding
请参阅[本指南](https://guides.codepath.com/android/RxJava-and-RxBinding)，了解如何使用Android视图实现RxJava。RxBinding库为许多标准Android视图实现Observable事件，使您可以更轻松地UI事件以利用基于响应的编程。
###RxLifcycle
还有一个名为[RxLifecycle](https://github.com/trello/RxLifecycle)的库，它为管理活动和片段的生命周期提供支持。在过去，RxAndroid通过AndroidObservables提供了这种支持，但是决定简化库。想了解更多信息，请参阅此[elease Note](https://github.com/ReactiveX/RxAndroid/releases/tag/v1.0.0)

RxLifcycle要求Activity都继承于RxActivity或者RxAppCompactActivity

###串联 Observables（这一段不好翻译啊┭┮﹏┭┮）
为了更好地理解订阅如何链接以及RxJava如何工作，最好首先了解进行subscribe（）调用时表面下发生的情况。在封面下创建订阅者对象。如果我们希望链接输入，可以使用各种运算符将一个订阅者类型映射到另一个订阅者类型。

了解更多信息，请观看[该视频](https://vimeo.com/144812843)
###相关链接
- https://www.captechconsulting.com/blogs/getting-started-with-rxjava-and-android
- http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/
- https://github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module
- http://saulmm.github.io/when-Iron-Man-becomes-Reactive-Avengers2/
- http://blog.stablekernel.com/replace-asynctask-asynctaskloader-rx-observable-rxjava-android-patterns/
- https://www.youtube.com/watch?v=_t06LRX0DV0/
- https://vimeo.com/144812843
- http://code.hootsuite.com/asynchronous-android-programming-the-good-the-bad-and-the-ugly/
- https://www.youtube.com/watch?v=va1d4MqLUGY&feature=youtu.be/
- http://www.slideshare.net/TimoTuominen1/rxjava-architectures-on-android-android-livecode-berlin/
- https://www.youtube.com/watch?v=va1d4MqLUGY&feature=youtu.be/
- https://speakerdeck.com/benjchristensen/reactive-streams-with-rx-at-javaone-2014
- http://www.philosophicalhacker.com/2015/06/12/an-introduction-to-rxjava-for-android/
- http://www.oreilly.com/programming/free/files/rxjava-for-android-app-development.pdf
- https://medium.com/@LiudasSurvila/droidcon-2015-london-part-1-698a6b750f30#.tvinpqa2q
- https://speakerdeck.com/passsy/get-reactive
- http://www.grokkingandroid.com/rxjavas-side-effect-methods/
- http://colintheshots.com/blog/?p=85/
Dan Lew's talk on common RxJava mistakes
- http://blog.danlew.net/2014/10/08/grokking-rxjava-part-4/#oldslowcode
- https://www.youtube.com/watch?v=k3D0cWyNno4