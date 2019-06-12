# Android线程：你所要知道的一切
 > 翻译自[https://www.toptal.com/android/android-threading-all-you-need-to-know](https://www.toptal.com/android/android-threading-all-you-need-to-know)
 
 每个Android开发人员都需要在他们的应用程序中处理线程。

在Android中启动应用程序时，它会创建第一个执行线程，称为“主”线程。主线程负责将事件分派到适当的用户界面小部件以及与Android UI工具包中的工具类进行通信。

为了使您的应用程序保持响应，必须避免使用主线程执行可能造成其卡顿的任何操作。

网络操作和数据库调用以及某些工具类的加载是人们应该在主线程中避免的操作的常见示例。当它们在主线程中被调用时，它们被同步调用，这意味着在操作完成之前UI将保持完全无响应。出于这个原因，它们通常在单独的线程中执行，从而避免在执行UI时阻塞UI（即，它们是从UI异步执行的）。

Android提供了许多创建和管理线程的方法，并且存在许多第三方库，使线程管理更加便捷。然而，有了这么多不同的方法，选择正确的方法可能会让人感到困惑。

在本文中，您将了解Android开发中的一些常见场景，其中线程变得至关重要，以及一些可应用于这些场景的简单解决方案等。

## Android中的线程
在Android中，您可以将所有线程工具类分为两个基本类别：

1. *附加到Activity/Fragment的线程*：这些线程与Activity/Fragment的生命周期相关联，并在Activity/Fragment被销毁后立即终止。
2. *未附加到任何Activity/Fragment的线程*：这些线程可以继续运行超出它们生成的Activity/Fragment（如果有）的生命周期。

### 附加到Activity/Fragment的线程工具类

#### AsyncTask
AsyncTask是Android里面最基本的一个有关线程的工具类。它使用简单，并且能够很好的处理一般场景。

示例：
```java
public class ExampleActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        new MyTask().execute(url);
    }

    private class MyTask extends AsyncTask<String, Void, String> {

        @Override
        protected String doInBackground(String... params) {
            String url = params[0];
            return doSomeWork(url);
        }

        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);
            // do something with result 
        }
    }
}
```
AsyncTask,但是，如果你需要延迟任务超出Activity/Fragment的生命周期则不适用。值得注意的是，即使像屏幕旋转这样简单的事情也会导致Activity销毁。
#### Loaders

Loaders是针对上述问题的解决方案。Loaders可以在Activity销毁时自动停止，也可以在重新创建Activity后自行重新启动。

Loader主要有两种类型：AsyncTaskLoader\CursorLoader，本篇文章主要介绍CursorLoader
AsyncTaskLoader与AsyncTask类似，但是要稍微复杂一点。
示例：
```java
public class ExampleActivity extends Activity{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        getLoaderManager().initLoader(1, null, new MyLoaderCallbacks());
    }
    
    private class MyLoaderCallbacks implements LoaderManager.LoaderCallbacks {

        @Override
        public Loader onCreateLoader(int id, Bundle args) {
            return new MyLoader(ExampleActivity.this);
        }

        @Override
        public void onLoadFinished(Loader loader, Object data) {

        }

        @Override
        public void onLoaderReset(Loader loader) {

        }
    }

    private class MyLoader extends AsyncTaskLoader {

        public MyLoader(Context context) {
            super(context);
        }

        @Override
        public Object loadInBackground() {
            return someWorkToDo();
        }
        
    }
}
```

#### Service（这部分就不翻译了，原文表达的不够完善 ）

### Android中的七种线程使用案例

#### 用户案例1：只是发起一个简单的网络请求，而不关心服务器返回结果
有时您可能希望向服务器发送API请求，而无需担心其响应。例如，您可能正在向应用程序的后端发送推送注册Token。由于这涉及通过网络发出请求，因此您应该从主线程以外的线程执行此操作。

##### 方式1：AsyncTask或者Loaders
您可以使用AsyncTask或Loader。但是，AsyncTask和加载器都依赖于Activity的生命周期。这意味着您需要等待执行调用并尝试阻止用户离开Activity，或者希望它在Activity被销毁之前执行。
##### 方式2：Service
Service可能更适合该场景，因为它不需要关联任何Activity。因此，即使在Activity被销毁之后，它也能够继续进行网络请求。另外，由于不需要服务器的响应，因此Service也不会受到限制。但是，由于Service在UI线程上运行，你仍然需要自己创建和管理线程。您还需要确保在网络请求完成后停止Service。

但是这样反而要做更多额外的工作。

##### 方式3：IntentService

我个人认为IntentService是最佳方案。

因为IntentService不需要依赖任何的Activity且运行在非UI线程，这方面完全匹配我们的需求。更好的是IntentService会自动停止，这样就不需要我们人工干预了。

#### 用户案例2：需要等待服务器返回的网络请求
这种用例更加常见一些。举个例子，你会通过调用一个服务端的api,拿到返回结果以后，展示在屏幕上

##### 方案1：Service或者 IntentService（也不翻译了）
##### 方案2：AsyncTask或者Loader(不翻译了)
##### 方案3：RXJAVA
你可能听说过RxJava，一个NetFlix开发的第三方库，在java中简直是魔法一样的存在。

在Android中我们可以使用RxAndroid库来使得处理异步任务变得简单。想要了解更多RxJava，点击[这里](https://www.toptal.com/android/functional-reactive-android-rxjava)

Rxjava提供了两个工具类：Observer和Subscriber

Observer 该工具类包含一些事务处理逻辑，它主要负责执行对应的事务处理并且返回对应的结果或者失败信息。

Subscriber 则是通过监听一个Observer,来接收Observer结果的一个工具类。

通过使用RxJava，你可以创建一个Observable

```java
Observable.create((ObservableOnSubscribe<Data>) e -> {
    Data data = mRestApi.getData();
    e.onNext(data);
})
```
一旦Observable创建成功，你就可以监听它。

可以使用RxAndroid库提供的相应函数，你可以通过observable选择对应的线程来执行你的事务逻辑，也可以选择具体的线程来返回结果。
```java
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread()
```
Schedulers则是在一个固定的线程里执行相应的事务处理。
AndroidSchedulers.mainThread()就是一个与主线程相关联的工具类。

鉴于我们调用的api为mRestApi.getData()，返回的结果为Data类型的对象，所以调用方式大致如下：
```java
Observable.create((ObservableOnSubscribe<Data>) e -> {
            try { 
                        Data data = mRestApi.getData();
                        e.onNext(data);
            } catch (Exception ex) {
                        e.onError(ex);
            }
})
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(match -> Log.i("rest api, "success"),
            throwable -> Log.e(“rest api, "error: %s" + throwable.getMessage()));
```
虽然没有使用RxJava的其他优势，你已经可以看到RxJava如何通过抽象线程的复杂性来编写更成熟的代码。

#### 用户案例3：链式网络调用

对于需要按顺序执行的网络调用（即，每个操作取决于前一操作的响应/结果），您需要特别注意生成意大利面条代码（难以维护，非结构化代码）。

比如，你请求的API需要首先获取一个Token

##### 方式1：AsyncTask或者loaders（不翻译了）
##### 方式2：使用RxJava的flatmap
在RxJava中，flatMap运算符从源observable中获取一个值，并返回另一个observable。您可以创建一个observable，然后使用第一个的返回结果创建另一个observable。

###### 步骤1：创建一个获取token的Observable
```java
public Observable<String> getTokenObservable() {
    return Observable.create(subscriber -> {
        try {
            String token = mRestApi.getToken();
            subscriber.onNext(token);

        } catch (IOException e) {
            subscriber.onError(e);
        }
    });
}
```
###### 步骤2：创建一个使用token获取数据的Observable
```java
public Observable<String> getDataObservable(String token) {
    return Observable.create(subscriber -> {
        try {
            Data data = mRestApi.getData(token);
            subscriber.onNext(data);

        } catch (IOException e) {
            subscriber.onError(e);
        }
    });
}
```
###### 步骤3：将两个Observable链接在一起，并订阅
```java
getTokenObservable()
.flatMap(new Function<String, Observable<Data>>() {
    @Override
    public Observable<Data> apply(String token) throws Exception {
        return getDataObservable(token);
    }
})
.subscribe(data -> {
    doSomethingWithData(data)
}, error -> handleError(e));
```

请注意，此方法的使用不仅限于网络调用;它可以处理需要在序列中但在不同线程上运行的任何操作集。

上面的所有用例都非常简单。线程之间的切换仅在每次完成任务后发生。更高级的场景 - 例如，两个或多个线程需要相互主动通信的场景 - 也可以通过这种方法得到支持。

#### 用户案例4：其他线程与UI线程的交互
考虑一种情况，你希望上传文件并在完成后更新用户界面。由于上传文件可能需要很长时间，因此无需让用户等待。您可以使用Service（可能还有IntentService）来实现此需求。但是，在这种情况下，更大的挑战是能够在文件上传（在单独的线程中执行）完成后在UI线程上调用方法。
##### 方案1:在Service中集成RxJava

RxJava本身或在IntentService内部可能并不理想。在订阅Observable时，您将需要使用基于回调的机制，并且构建IntentService以执行简单的同步调用，而不是回调。

另一方面，使用Service，您需要手动停止Service，这需要更多的工作。
##### 方案2：BroadcastReceiver
Android提供此工具类，可以监听全局事件（例如，电池事件，网络事件等）以及自定义事件。您可以使用此工具类创建上载完成时触发的自定义事件。为此，您需要创建一个扩展BroadcastReceiver的自定义类，在manifest中注册它，并使用Intent和IntentFilter创建自定义事件。要触发事件，需要调用sendBroadcast方法。

*Manifest*
```xml
<receiver android:name="UploadReceiver">
   
    <intent-filter>
        <action android:name="com.example.upload">
        </action>
    </intent-filter>
   
</receiver>
```
*Receiver*
```java
public class UploadReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getBoolean(“success”, false) {
            Activity activity = (Activity)context;
            activity.updateUI();
    }

}
```
*Sender*
```java
Intent intent = new Intent();
intent.setAction("com.example.upload"); 
sendBroadcast(intent);
```
这种方法是可行的选择。但正如你所注意到的，它涉及一些工作，太多的广播可能会减慢响应速度。
##### 方案3：使用Handler
Handler是一个可以附加到线程然后通过简单消息或Runnable任务在该线程上执行某些操作的工具类。它与另一个工具类Looper一起工作，Looper负责特定线程中的消息处理。

创建Handler时，它可以在构造函数中获取Looper对象，该对象指示handler附加到哪个线程。如果要使用附加到主线程的handler，则需要通过调用Looper.getMainLooper（）来使用与主线程关联的looper。在这种情况下，要从后台线程更新UI，你可以创建附加到UI线程的handler，然后将所需要做的操作封装成一个*runnable*传递给handler：
```java
Handler handler = new Handler(Looper.getMainLooper());
    handler.post(new Runnable() {
        @Override
        public void run() {
            // update the ui from here                
        }
    })
```
这种方法比第一种方法好很多，但有一种更简单的方法可以做到这一点......
##### 方案4：使用EventBus
EventBus是GreenRobot的一个受欢迎的开源库，它使工具类能够安全地相互通信。由于我们的使用场景是我们只想更新UI，因此这可能是最简单和最安全的选择。
###### 步骤1：创建一个Event类，例如UIEvent
###### 步骤2：监听该event
```java
@Subscribe(threadMode = ThreadMode.MAIN)  
public void onUIEvent(UIEvent event) {/* Do something */};

 register and unregister eventbus : 

@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
    super.onStop();
    EventBus.getDefault().unregister(this);
}
```
###### 步骤3：发送Event：*EventBus.getDefault().post(new UIEvent());*
使用注释中的ThreadMode参数，您将指定要为此事件订阅的线程。在我们这里的示例中，我们选择主线程，因为我们希望事件的接收者能够更新UI。您可以根据需要构建UIEvent类以包含其他信息。

在这Services中逻辑如下
```java
class UploadFileService extends IntentService {
    // …
    Boolean success = uploadFile(File file);
    EventBus.getDefault().post(new UIEvent(success));
    // ...
}
```
在activity/fragment中逻辑如下：
```java
@Subscribe(threadMode = ThreadMode.MAIN)  
public void onUIEvent(UIEvent event) {//show message according to the action success};
```
使用EventBug库，让线程之间交互变得更加简单。

#### 用户案例5：线程之间基于用户操作的双向通信

假设您正在创建一个mediaplayer，并且您希望它能够在应用程序屏幕关闭时继续播放音乐。在这种情况下，您将希望UI能够与media播放线程通信（例如，播放，暂停和其他操作），并且还希望media线程基于某些事件（例如，错误，缓冲状态）更新UI等。

完整的媒体播放器示例超出了本文的范围。但是，你可以在[这里](https://github.com/googlesamples/android-UniversalMusicPlayer)和[这里](https://github.com/google/ExoPlayer)找到很好的教程。

##### 方案1：使用EventBus
你可以在这里使用EventBus。但是，从UI线程发布事件并在Service中接收事件通常是不安全的。这是因为你在发送消息时无法知道Service是否正在运行。
##### 方案2：使用BoundService
BoundService是绑定到activity/fragment的service。这意味着activity/fragment始终知道service是否正在运行，此外，它还可以访问service的公共方法。

要实现它，您需要在service中创建自定义Binder并创建一个返回service的方法。
```java
public class MediaService extends Service {

    private final IBinder mBinder = new MediaBinder();

    public class MediaBinder extends Binder {
        MediaService getService() {
            // Return this instance of LocalService so clients can call public methods
            return MediaService.this;
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

}
```
要将activity绑定到Service，您需要实现ServiceConnection，它是监视Service状态的类，并使用bindService方法进行绑定：
```java
// in the activity
MediaService mService;
// flag indicates the bound status
boolean mBound;

@Override
    protected void onStart() {
        super.onStart();
        // Bind to LocalService
        Intent intent = new Intent(this, MediaService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    private ServiceConnection mConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName className,
                IBinder service) {

            MediaBinder binder = (MediaBinder) service;
            mService = binder.getService();
            mBound = true;
        }

        @Override
        public void onServiceDisconnected(ComponentName arg0) {
            mBound = false;
        }
    };
```
[这里](https://developer.android.com/guide/components/bound-services.html)你可以看到更全的示例代码

要在用户点击“播放”或“暂停”按钮时与服务进行通信，您可以绑定到该服务，然后在服务上调用相关的public方法。当存在媒体事件并且您想要将其传回Activity/Fragment时，您可以使用之前的一些方案（例如，BroadcastReceiver，Handler或EventBus）。
#### 用户案例6：并行执行操作来获得结果

假设您正在编写一个旅游应用程序，并且您希望在从多个来源（不同的数据提供者）获取的显示在地图上。由于并非所有源都可靠，您可能希望忽略失败的源并继续渲染地图。为了并行处理这些操作，每个API调用必须在不同的线程中进行。

##### 方案1：使用RxJava
在RxJava中，您可以使用merge（）或concat（）运算符将多个observable组合成一个。然后，您可以订阅“合并”的observeable并等待所有结果返回。但是，这种方法不能按我们所预期的运行，如果一个API调用失败，则合并的observable将会整体失败。

##### 方案2：使用Java自带的工具类

Java中的ExecutorService可以创建一个固定（可配置）数目的线程，并同时执行任务。该服务会返回一个Future对象，该对象最终通过invokeAll（）方法返回所有结果。您发送到ExecutorService的每个任务都应该包含在Callable接口中，该接口是用于创建可以引发异常的任务的接口。从invokeAll（）获得结果后，您可以检查每个结果并相应地继续。例如，假设您有三种景区类型来自三个不同的渠道，并且您希望进行三次并行调用：
```java
ExecutorService pool = Executors.newFixedThreadPool(3);
    List<Callable<Object>> tasks = new ArrayList<>();
    tasks.add(new Callable<Object>() {
        @Override
        public Integer call() throws Exception {
            return mRest.getAttractionType1();
        }
    });

    // ...

    try {
        List<Future<Object>> results = pool.invokeAll(tasks);
        for (Future result : results) {
        try {
            Object response = result.get();
            if (response instance of AttractionType1... {}
            if (response instance of AttractionType2... {}
                ...
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
```
这样，您就可以并行运行所有操作。因此，您可以单独检查每个操作中的错误，并根据需要忽略单个请求出错的问题。这种方法比使用RxJava更容易。相比较来说，它更简单，更短，并且不会因为一个请求异常而导致所有操作失败。
#### 用户案例7：查询本地的sqlite数据
在操作本地SQLite数据库时，建议从后台线程使用数据库，因为数据库调用（特别是大型数据库或复杂查询）可能非常耗时，导致阻塞UI线程。查询SQLite数据时，会得到一个Cursor对象，然后可以使用该对象来获取实际数据。
```java
Cursor cursor = getData();
String name = cursor.getString(<colum_number>);
```
##### 方案1：使用RxJava
你可以使用RxJava从数据库中获取数据，就像我们从后端获取数据一样：
```java
public Observable<Cursor> getLocalDataObservable() {
    return Observable.create(subscriber -> {
        Cursor cursor = mDbHandler.getData();
        subscriber.onNext(cursor);
    });
}
```
您可以使用getLocalDataObservable（）返回的observable，如下所示：
```java
getLocalDataObservable()
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
 .subscribe(cursor -> String name = cursor.getString(0),
                   throwable -> Log.e(“db, "error: %s" + throwable.getMessage()));
```
虽然这当然是一种很好的方法，但有一种方法甚至更好，因为有一个工具类专为这种情况而生。
##### 方案2：使用CursorLoader + ContentProvider
Android提供了CursorLoader，这是一个用于加载SQLite数据和管理相应线程的原生类。它是一个返回Cursor的Loader，我们可以通过调用getString（），getLong（）等简单方法来获取数据。
```java
public class SimpleCursorLoader extends FragmentActivity implements
LoaderManager.LoaderCallbacks<Cursor> {

  public static final String TAG = SimpleCursorLoader.class.getSimpleName();
  private static final int LOADER_ID = 0x01;
  private TextView textView;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_cursor_loader);
    textView = (TextView) findViewById(R.id.text_view);
    getSupportLoaderManager().initLoader(LOADER_ID, null, this);

  }

  public Loader<Cursor> onCreateLoader(int i, Bundle bundle) {

    return new CursorLoader(this,
      Uri.parse("content://com.github.browep.cursorloader.data")
      , new String[]{"col1"}, null, null, null);
    }

    public void onLoadFinished(Loader<Cursor> cursorLoader, Cursor cursor) {
      if (cursor != null && cursor.moveToFirst()) {
        String text =  textView.getText().toString();
        while (cursor.moveToNext()) {
          text += "<br />" + cursor.getString(1);
          cursor.moveToNext();
        }
        textView.setText(Html.fromHtml(text) );
      }
    }

    public void onLoaderReset(Loader<Cursor> cursorLoader) {
      
    }

}
```
CursorLoader与ContentProvider工具类一起使用。该工具类提供了大量的实时数据库功能（例如，更改通知，触发器等），使开发人员能够更轻松地实现更好的用户体验。
## 在Android中没有针对线程的完美解决方案
Android提供了许多处理和管理线程的方法，但它们都不是能够解决所有问题。根据您的使用场景选择正确的线程方法可以使整个解决方案易于实现和理解。原生工具类适合某些场景，但并非适用于所有情况。这同样适用于花哨的第三方解决方案。在你的下一个Android项目时希望能够本文对你有所帮助。