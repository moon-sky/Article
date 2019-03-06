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






