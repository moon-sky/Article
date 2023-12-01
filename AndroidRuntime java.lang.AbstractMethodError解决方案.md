### AndroidRuntime: java.lang.AbstractMethodError解决方案

#### 背景介绍

今天同事尝试编译apk的release版本，编译成功，但是运行时，却爆出这个运行时异常，导致crash。

#### 排查过程

##### 定位出错位置

通过查看异常的堆栈，爆出异常的方法是一个我们引入的jar包里，而这jar包引用方法是compileOnly，而compileOnly依赖的用途

1. 编译时用于构建项目，但是运行时不需要
2. 在编译时依赖它的api，但是它API的真正实现是在运行时环境

我们具体看一下这个AbstractMethod异常的api解释

>```java
>/**
> * Thrown when an application tries to call an abstract method.
> * Normally, this error is caught by the compiler; this error can
> * only occur at run time if the definition of some class has
> * incompatibly changed since the currently executing method was last
> * compiled.
>```

大体意思就是这个错误只有在运行时才会出现，编译时与运行时的方法不一致导致。

另外一个关键信息是，debug版本就没有这个错误，而debug与release的主要区别就在于在build.gradle中是否将minifyenabled置为true,这个开关的主要用途就是是否启用混淆。

##### 尝试解决

我们先从混淆入手，把这调用该方法的类加上keep class，然后重新打release包，运行一下，果然没有再报那个错误了。

#### 总结

当我们使用一些jar包只用来编译时使用时，如果我们加了混淆，会导致运行时系统类回调时找不到这个对应的实现类，进而报错





































































































