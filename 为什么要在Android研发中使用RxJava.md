##为什么要在Android研发 中使用RxJava
>【翻译】原文链接[https://medium.com/@lpereira/why-should-we-use-rxjava-on-android-c9066087c56c](https://medium.com/@lpereira/why-should-we-use-rxjava-on-android-c9066087c56c)

>如有侵权，请立即告知

![image](https://cdn-images-1.medium.com/max/800/1*WFPcM9DD3O9EhoC2aOWMuA.png)

Reactive Extensions（Rx）是一组接口和方法，它们为开发人员提供了一种快速解决问题，简单维护和易于理解的方法。RxJava提供了一套工具，可以帮助您编写简洁明了的代码。

老实说，起初我认为RxJava太难理解了，为了使用这些新工具而添加库的想法对我来说非常麻烦。过了一段时间，我意识到我过去一直在一遍又一遍地做着相同的实现逻辑实现来努力让用户了解应用程序的最新动态。

我所做的工作只是因为有了新的要求，甚至是因为信息的呈现方式或处理方式，就重构了所有必要的方法和接口，这简直太荒谬了。此外，去了解别人写的大量代码，就是一种痛苦。

让我们想象一下，我们需要从数据库中获取用户列表，然后在我们的视图中显示这些结果。为此，我们可以创建一个AsyncTask来从数据库中获取所有信息，然后将其插入到我们的适配器中，以将结果呈现给用户。出于本示例的目的，我们写一个简单任务：
```java
public class SampleTask extends AsyncTask<Void,Void,List<Users>> {
    private final SampleAdapter mAdapter;

    public SampleTask(SampleAdapter sampleAdapter) {
        mAdapter = sampleAdapater;
    }

    @Override
    protected List<Users> doInBackground(Void... voids) {
        //fetch there results from the database and return them to the onPostExecute
        List<Users> users = getUsersFromDatabase();
        return users;
    }

    @Override
    protected void onPostExecute(List<Users> users) {
        super.onPostExecute(products);
        // Checking if there are users on the database
        if(users == null) {
            //No users, presenting a view saying there are no users
            showEmptyUsersMessageView();
            return;
        }

        mAdapter.addAll(users);
        mAdapter.notifyDataSetChanged();
    }
}
```
如果有新要求说只应出现非Guest用户，那么应该是在将数据添加到适配器加以处理或更改数据库查询加条件。此外，想象一下，如果不是只从用户那里获取信息，而是现在要求你需要从数据库中获取其他东西以呈现在同一个适配器上呢？

这就是为什么我们要用RxJava，它可以在这些情况下帮助我们很多。例如，我们可以像以下代码这样，更好地组织我们的代码：
```java
public Observable<List<User>> fetchUsersFromDatabase() {
    return Observable.create(new Observable.OnSubscribe<List<User>(){
        @Override
        public void call(Subscriber<? super List<User>> subscriber){
            // Fetch information from database
            subscriber.onNext(getUserList());
            subscriber.onCompleted();
        }
    });
}
```
然后你可以这样调用它
```java
fetchUsersFromDatabase()
    //will process everything in a new thread
    .subscribeOn(Schedulers.io())
    //will listen the results on the main thread
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Subscriber<List<User>>() {
        @Override
        public void onCompleted() {
    
        }
    
        @Override
        public void onError(Throwable e) {
    
        }
    
        @Override
        public void onNext(List<User> users) {
            //Do whatever you want with each user
        }
    });
}
```
你一定在思考，如果我们想得到除Guest以外的所有用户？使用这种方法很容易改变它，开发人员只需要添加一个过滤器。
```java
fetchUsersFromDatabase()
        .filter(new Func1<User, Boolean>() {
            @Override
            public Boolean call(User user) {
                //only return the users which are not guests
                return !user.isGuest();
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<User>() {
                       @Override
                       public void onCompleted() {
                       }

                       @Override
                       public void onError(Throwable e) {
                           /*Check if there was any error while retrieving from database*/
                       }

                       @Override
                       public void onNext(User user) {
                           //Do whatever you want with each user
                       }
                   }
        );
```
一个从数据库转换或过滤信息的简单操作，却需要新的接口和重构代码以便与已有的体系结构保持一致。而使用RxJava可以轻松实现，只需创建一个Observable来检索所有信息，然后您可以使用这些[方法](https://github.com/ReactiveX/RxJava/wiki/Filtering-Observables)中的任何一种来过滤和检索所需的信息。

虽然你一定会说不错，可读性和代码接口狗看起来也更好，但这种方法产生了如此多的代码。嗯，你是对的，但这里就是需要Retrolambda出场了。该库提供了使用Java 8 lambda表达式，方法引用等的兼容性。

使用此库将有助于显著减少代码：
```java
fetchUsersFromDatabase()
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(value -> {
                    //Do whatever with the value
                },error -> {
                    //do something with in case of error
                }
        );
```
同样你会问，如果我们需要运行其他查询同时也在同一个适配器上显示它，应该怎么做？那么，我们可以这样做：
```java
fetchUsersFromDatabase()
        .zipWith(fetchSomethingElseFromDatabase(), (users, somethingElse) -> {/*here combine users and something else into a new object*/
        })
        .subscribe( o -> {/*use the combine object from users and something else to fill the adapter */}
```
在这个例子中，我们将用户数据和从数据库中查询的其他数据一起加入到结果中，然后我们将数据传入适配器。更容易维护，更少的代码和易于阅读对吗？

为了更深入地了解可以检索信息的方法还有可用的函数，下面这边文章，它应该对您有所帮助。此外，它可以帮助您了解这相关的一切信息。

[文章传送门](Party tricks with RxJava, RxAndroid & Retrolambda)

另外，这个教程帮助我提高了我的RxJava技能，希望你也能够看到它，它应该会对你有所帮助。
