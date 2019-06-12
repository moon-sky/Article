## RxJava官方Readme中文翻译版（个人翻译）
>最近一直在看RxJava的资料，也希望通过翻译这些相关文章，达到了解的程度。如果大神们觉得哪里翻译的不好，千万别喷我，毕竟我的初衷也只是让自己看懂（手动滑稽）。
先附上正版[ReadMe链接](https://github.com/ReactiveX/RxJava)

### RxJava:面向JVM的响应式扩展
RxJava是Reactive Extensions的Java VM实现：用于通过使用可观察序列来编写异步和基于事件的程序的一个库。

它扩展了观察者模式以支持数据/事件序列，并添加了允许以声明方式组合序列的运算符，同时抽象出对低级线程，同步，线程安全和并发数据结构等问题的关注。
##### Version 2.x（[Javadoc](http://reactivex.io/RxJava/2.x/javadoc/)）
- 单一依赖:[Reactive-Streams](https://github.com/reactive-streams/reactive-streams-jvm)
- 继续支持Java 6+和Android 2.3+
- 通过1.x周期和[Reactive-Streams-Commons](https://github.com/reactor/reactive-streams-commons)研究项目学到的设计变更带来的性能提升。
- Java 8 lambda-友好的API
- 关于并发源（线程，池，事件循环，fiber，Actors等）的不同意见
- 异步或同步执行
- 参数化并发的虚拟时间和调度程序

版本2.x和1.x将并存数年。它们将具有不同的组ID（io.reactivex.rxjava2 vs io.reactivex）和命名空间（io.reactivex vs rx）。

请参阅wiki文章,了解版本1.x和2.x之间的差异[What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)。在[Wiki Home](https://github.com/ReactiveX/RxJava/wiki)上了解有关RxJava的更多信息。
##### Version 1.x
1.x版本截止日期为2018年3月31日。不会进行进一步的开发，支持，维护，PR和更新。最后一个版本1.3.8的Javadoc目前仍然可以访问。

#### 开始上手
##### 依赖搭建
第一步是将RxJava 2包含到您的项目中，例如，使用Gradle编译依赖项：
```java
implementation "io.reactivex.rxjava2:rxjava:2.x.y"
```
（请用最新的版本号替换x和y :[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxjava/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxjava))
##### Hello World
第二步就是写一个Hello World的程序
```java
package rxjava.examples;

import io.reactivex.*;

public class HelloWorld {
    public static void main(String[] args) {
        Flowable.just("Hello world").subscribe(System.out::println);
    }
}
```
如果你对平台还不支持 java 8的lambdas语法。你可以认为的创建一个内部Consumer类

```java
import io.reactivex.functions.Consumer;

Flowable.just("Hello world")
  .subscribe(new Consumer<String>() {
      @Override public void accept(String s) {
          System.out.println(s);
      }
  });
```
##### 基类
以下是Rxjava2的一些基类:
（流动-在我看来 就是一次处理过程）

- [io.reactivex.Flowable](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html): 0-N次流动，支持Reactive-Streams 和背压
- [io.reactivex.Observable](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html): 0-N次流动，不支持背压
- [io.reactivex.Single](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Single.html):只处理一个item或者回调一个错误，只有一次流动
- [io.reactivex.Completable](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Completable.html):只是代表一次完成或者失败 ，没有任何值
- [io.reactivex.Maybe](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Maybe.html):一个item或者一个错误 回调或者没有item

##### 专业术语
###### Upstream, downstream
RxJava中的数据流由源，零个或多个中间步骤组成，后跟数据使用者或组合器步骤（负责通过某种方式使用数据流）：
```java
source.operator1().operator2().operator3().subscribe(consumer);

source.flatMap(value -> source.operator1().operator2().operator3());
```
在这里，如果我们将自己想象在operator2上，向左看向源，则称为上游。向订户/消费者的右侧看，称为下游。当每个元素写在一个单独的行上时，这通常更明显：
```java
source
  .operator1()
  .operator2()
  .operator3()
  .subscribe(consumer)
```
##### 运动的对象
在RxJava的文档中，emission, emits, item, event, signal, data和message被视为同义词，表示沿数据流传播的对象。
##### 背压
当数据流通过异步步骤时，每个步骤可以以不同的速度执行不同的操作。为了避免压垮这些步骤，这些步骤通常表现为由于临时缓冲或需要跳过/丢弃数据而增加的内存使用量，所以应用所谓的背压，这是流量控制的一种形式，其中步骤可以表达多少item准备好了。这允许在通常无法知道上游将向其发送多少项的步骤的情况下约束数据流的存储器使用。

在RxJava中，专用的Flowable类被指定为支持背压，而Observable专用于非背压操作（短序列，GUI交互等）。其他类型，Single，Maybe和Completable不支持背压，也不应该支持;总有临时存储一个item的空间。

##### 整合时间
指的是在使用各种中间操作符准备数据流的时间就称为整合时间（Assembly time）
```java
Flowable<Integer> flow = Flowable.range(1, 5)
.map(v -> v * v)
.filter(v -> v % 3 == 0)
;
```
此时，数据尚未流动，并且没有发生任何事情。

##### 订阅时间
当在内部建立处理步骤链的流上调用subscribe（）时，这是一个临时状态：
```java
flow.subscribe(System.out::println)
```
这是触发订阅的时间（请参阅doOnSubscribe）。某些来源在此状态下阻塞或开始分发item。
##### 运行时
这是流主动发出item，错误或完成信号时的状态：
```java
Observable.create(emitter -> {
     while (!emitter.isDisposed()) {
         long time = System.currentTimeMillis();
         emitter.onNext(time);
         if (time % 2 != 0) {
             emitter.onError(new IllegalStateException("Odd millisecond!"));
             break;
         }
     }
})
.subscribe(System.out::println, Throwable::printStackTrace);
```
实际上，这是在上面给出的例子的主体执行的时候。

##### 简单的后台计算
RxJava的一个常见用例是在后台线程上运行一些计算，网络请求并在UI线程上显示结果（或错误）：
```java
import io.reactivex.schedulers.Schedulers;

Flowable.fromCallable(() -> {
    Thread.sleep(1000); //  imitate expensive computation
    return "Done";
})
  .subscribeOn(Schedulers.io())
  .observeOn(Schedulers.single())
  .subscribe(System.out::println, Throwable::printStackTrace);

Thread.sleep(2000); // <--- wait for the flow to finish
```
这种链式方法称为流畅的API，类似于构建器模式。但是，RxJava的响应类型是不可变的;每个方法调用都返回一个带有添加行为的新Flowable。为了说明，该示例可以重写如下：
```java
Flowable<String> source = Flowable.fromCallable(() -> {
    Thread.sleep(1000); //  imitate expensive computation
    return "Done";
});

Flowable<String> runBackground = source.subscribeOn(Schedulers.io());

Flowable<String> showForeground = runBackground.observeOn(Schedulers.single());

showForeground.subscribe(System.out::println, Throwable::printStackTrace);

Thread.sleep(2000);
```

通常，您可以通过subscribeOn将计算或阻止IO移动到其他线程。数据准备就绪后，您可以确保通过observeOn在前台或GUI线程上处理它们。

##### 调度程序

RxJava运算符不能直接使用Threads或ExecutorServices，而是使用所谓的调度程序来抽象统一API背后的并发源。RxJava 2具有几个可通过Scheduler实用程序类访问的标准调度程序。

- Schedulers.computation（）：在后台运行固定数量的专用线程上的计算密集型工作。大多数异步操作符使用它作为其默认调度程序。
- Schedulers.io（）：在动态变化的线程集上运行类I / O或阻塞操作。
- Schedulers.single（）：以顺序和FIFO方式在单个线程上运行。
- Schedulers.trampoline（）：在其中一个参与线程中以顺序和FIFO方式运行，通常用于测试目的。

这些可在所有JVM平台上使用，但某些特定平台（如Android）具有自己的典型调度程序：AndroidSchedulers.mainThread（），SwingScheduler.instance（）或JavaFXSchedulers.gui（）。

此外，还可以选择通过Schedulers.from（Executor）将现有的Executor（及其子类型，如ExecutorService）包装到Scheduler中。例如，这可以用于具有更大但仍然固定的线程池（与calculate（）和io（）不同）。

最后的Thread.sleep（2000）不是偶然的。在RxJava中，默认调度程序在守护程序线程上运行，这意味着一旦Java主线程退出，它们都会停止并且后台计算可能永远不会发生。在此示例情况下休眠一段时间，您可以在控制台上查看流的输出，并留出时间。


##### 流的并发
RxJava中的流本质上是顺序分割为可以彼此同时运行的处理阶段：
```java
Flowable.range(1, 10)
  .observeOn(Schedulers.computation())
  .map(v -> v * v)
  .blockingSubscribe(System.out::println);
```
此示例流在计算调度程序上将数字从1到10求平方，并在“主”线程上使用结果（更准确地说，是blockingSubscribe的调用者线程）。但是，lambda v  - > v * v不会并行运行此流程;它一个接一个地在同一个计算线程上接收值1到10。


##### 并行处理
并行处理数字1到10涉及更多：

```java
Flowable.range(1, 10)
  .flatMap(v ->
      Flowable.just(v)
        .subscribeOn(Schedulers.computation())
        .map(w -> w * w)
  )
  .blockingSubscribe(System.out::println);
```

实际上，RxJava中的并行性意味着运行独立流并将其结果合并回单个流。运算符flatMap通过首先将1到10中的每个数字映射到它自己的单个Flowable，运行它们并合并计算结果的平方来完成此操作。

但请注意，flatMap不保证任何顺序，内部流的最终结果可能最终交错。还有其他操作符：

- concatMap，一次映射并运行一个内部流程
- concatMapEager“一次”运行所有内部流，但输出流将按照创建内部流的顺序。

##### 依赖衍生流

flatMap是一个功能强大的运算符，可以在很多情况下提供帮助。例如，给定一个返回Flowable的服务，我们想要使用第一个服务发出的值调用另一个服务：
```java
Flowable<Inventory> inventorySource = warehouse.getInventoryAsync();

inventorySource.flatMap(inventoryItem ->
    erp.getDemandAsync(inventoryItem.getId())
    .map(demand 
        -> System.out.println("Item " + inventoryItem.getName() + " has demand " + demand));
  )
  .subscribe();
```
#### 依赖
有时，当一个item可用时，人们希望对其执行一些依赖计算。这有时被称为延续，并且取决于应该发生什么以及涉及什么类型，可能涉及各种操作员来完成。

##### 依赖
最典型的情况是给出一个值，调用另一个服务，等待并继续其结果：

```java
service.apiCall()
.flatMap(value -> service.anotherApiCall(value))
.flatMap(next -> service.finalCall(next))
```

通常情况下，后面的序列也需要来自前面Map中的值。这可以通过将外部flatMap移动到前一个flatMap的内部部分来实现，例如：

```java
service.apiCall()
.flatMap(value ->
    service.anotherApiCall(value)
    .flatMap(next -> service.finalCallBoth(value, next))
)
```
这里，value将在内部flatMap中可用，由lambda变量捕获提供。

##### 无依赖
在其他场景中，第一个源/数据流的结果是无关紧要的，并且人们希望继续使用准独立的另一个源。这里，flatMap也可以

```java
Observable continued = sourceObservable.flatMapSingle(ignored -> someSingleSource)
continued.map(v -> v.toString())
  .subscribe(System.out::println, Throwable::printStackTrace);
```
但是，在这种情况下，继续使用Observable而不是可能更合适的Single。（这是可以理解的，因为从flatMapSingle的角度来看，sourceObservable是一个多值源，因此映射也可能导致多个值）。

通常使用Completable作为介体及其运算符，然后继续使用其他东西，这种方式更具表现力（以及更低的开销）：
```java
sourceObservable
  .ignoreElements()           // returns Completable
  .andThen(someSingleSource)
  .map(v -> v.toString())
```
sourceObservable和someSingleSource之间唯一的依赖关系是前者应该正常完成，以便消耗后者。

##### 递延依赖
有时，前一个序列和新序列之间存在隐含的数据依赖性，由于某种原因，它不会流经“常规通道”。人们倾向于像下面这样写：

```java
AtomicInteger count = new AtomicInteger();

Observable.range(1, 10)
  .doOnNext(ignored -> count.incrementAndGet())
  .ignoreElements()
  .andThen(Single.just(count.get()))
  .subscribe(System.out::println);
```
不幸的是，这会打印0，因为Single.just（count.get（））是在汇编时计算的当时数据流尚未运行。我们需要一些推迟评估此Single源的内容直到运行时也就是main source 完成时：
```java
AtomicInteger count = new AtomicInteger();

Observable.range(1, 10)
  .doOnNext(ignored -> count.incrementAndGet())
  .ignoreElements()
  .andThen(Single.defer(() -> Single.just(count.get())))
  .subscribe(System.out::println);
```
或者
```java
AtomicInteger count = new AtomicInteger();

Observable.range(1, 10)
  .doOnNext(ignored -> count.incrementAndGet())
  .ignoreElements()
  .andThen(Single.fromCallable(() -> count.get()))
  .subscribe(System.out::println);
```
#### 类型转换







