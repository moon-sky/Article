### Android待研究技术 ###
1. DataBinding
2. RecycleView
3. HandlerThread
4. Android混淆技巧与反编译[http://pan.baidu.com/s/1eQ8k85G](http://pan.baidu.com/s/1eQ8k85G)
5. Dex guard

### Android开发技巧 ###
1. handerl.obtainMessage(what) 该方法省去了 实例化一个Message对象。看源码可以发现

    public static Message obtain(Handler h, int what) {
    Message m = obtain();
    m.target = h;
    m.what = what;
    
    return m;
    }