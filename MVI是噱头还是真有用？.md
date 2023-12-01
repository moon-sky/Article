### MVI?是噱头还是真有用？

今天逛技术论坛的时候偶然发现了这个词，眼前一亮，因为之前活跃在大众面前的架构模式是MVC、MVP、MVVM，至于MVI真的是第一次看到，心里一慌，自己这么out了吗？赶紧Google一下。

不看不知道，关于MVI的文章还是不少的，甚至github上有开源的帮助开发者用MVI模式构建项目的代码。并且MVI的概念在2015年的时候就被提出来，但是一些文章也是最近两年才开始写的，由此可见一项新技术或者概念从提出到走到大众面前确实需要很长的路要走。

我会在文件末尾放上一些我觉得写的还不错的文章，下面我就简单说一下个人对于MVI模式的理解，纯粹是总结一下自己的学习心得，不好误喷。

#### MVI是什么？

先来一波概念解释，MVI即Model-View-Intent

**Model** 这里的Model与其他架构模式的M不太一样，它在这里代表不仅仅是代表数据的存储获取还包括了---状态State，界面或者view控件都有不同的状态，例如列表有empty状态、loading状态、fail状态、success状态，登录界面有logining状态、fail状态、success跳转状态

**View** 这里跟其他模式一样，都是代表界面或者view控件

**Intent** 意图？此意图非彼意图，它是一种用户行为的抽象，在我看来更像是具有Type的Event，也有文章把它理解为action，其实本质上都是一样的。比如用户点击刷新按钮的行为，我们可以定义为RefreshIntent

接下来我们再来一波图示

![Android MVI with Kotlin Coroutines &amp; Flow | QuickBird Studios Blog](https://gitee.com/moonsky/image-bed/raw/master/MVI_better.png)

大致就是这个意思了。

#### 为啥用它？

(⊙o⊙)…看你个人爱好吧，没有最好的架构。

相对于其他的架构模式来说，在我看来它更加符合面向过程编程。

在这个模式里，数据流向是单向的，并且状态是不可变，所以方便我们追踪业务的过程，定位问题，并且view的变化是完全依赖于state的监听，与数据完全解耦（当然这点跟MVVM好像没有啥差别)。这里应该是利用了一个职责分离的感念，让原本臃肿的viewmodel拆分出Intent以及State让开发更加清晰聚焦。我觉得对于界面不是很复杂，业务逻辑较少的界面可以尝试使用，如果是太复杂的话，需要写很多state和intent，会产生很多模板代码，当然这点也可以通过其他方式来优化。

#### 怎么用？

这里有一个图，大家看一下就明白了

ViewEvent对应的就是Intent，而Model对应的是ViewState或者ViewEffect

![img](https://gitee.com/moonsky/image-bed/raw/master/1*ohFythvIKvgVUy_08dF4Ag.png)

#### 总结

又是一篇比较水的文章😅，希望大家以后多交流，如果需要可以加我QQ511014405

#### 推荐文章

[Android MVI-Reactive Architecture Pattern](https://abhiappmobiledeveloper.medium.com/android-mvi-reactive-architecture-pattern-74e5f1300a87)

[【Android】MVI架构快速入门：从双向绑定到单向数据流](https://blog.csdn.net/vitaviva/article/details/109406873)

