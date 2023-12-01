# 优雅的获取Application Context

### 前言

在android开发中，很多场景我们都会用到Context，比如注册BroadcastReceiver，获取系统service，获取资源等。但是由于Context一般与生命周期关联，错误使用容易造成内存泄漏，所以我们一般会选择使用生命周期最长的Application Context，在其他类中获取application context的方式有两种

第一种方式：

在application添加一个context类型的成员变量，通过Application.getContext方法获取

```kotlin
class MyApplication: Application() {
    companion object{
        lateinit var mContext: Context
    }

    override fun onCreate() {
        super.onCreate()
        mContext=applicationContext
    }

}
class TestContext{
    fun getAppContext(){
        MyApplication.mContext
    }

}
```

第二种方式：

通过传入context类型的形参，再通过context.getApplicationContext方法获取application context

``` kotlin
    fun getAppContextM2(context: Context){
        context.applicationContext
    }
```

但是以上两种方式，也有明显的缺点，第一种你必须要自己创建一个application，然后编写静态方法。第二种，需要注入context，耦合性较高，并且容易造成内存泄漏



### 优化方式

```kotlin
object ContextHelper{
    fun getAppContext():Application?{
        var app:Application?
        try {
            app= Class.forName("android.app.AppGlobals").getMethod("getInitialApplication").invoke(null) as Application
        }catch (e:Exception){
            app= Class.forName("android.app.ActivityThread").getMethod("currentApplication").invoke(null) as Application
        }
        return app

    }
}
```

这里利用了反射的方式，调用一些隐藏方法，可以有效的解耦，并且不用注入带来的生命周期问题。