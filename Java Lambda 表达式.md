##Java Lambda 表达式
[翻译]原文链接[http://tutorials.jenkov.com/java/lambda-expressions.html#single-method-interface](http://tutorials.jenkov.com/java/lambda-expressions.html#single-method-interface)

- Java Lambdas和单一方法接口
    - 匹配Lambda到接口
    - 具有默认和静态方法的接口
- Lambda表达式与匿名接口实现
- Lambda类型推断
- Lambda参数
    -  零参数
    -  一个参数
    -  多个参数
    -  参数类型
    -  来自Java 11的var参数类型
-  Lambda函数
-  从Lambda表达式返回值
-  Lambdas作为对象
-  访问变量
    -  访问局部变量
    -  访问实例变量
    -  访问静态变量
-  方法引用
    -  静态方法引用
    -  参数方法引用
    -  实例方法引用
    -  构造函数引用

Java lambda表达式是Java 8中的新增功能.Java lambda表达式是Java进入函数式编程的第一步。
因此，Java lambda表达式是一个可以在不属于任何类的情况下创建的函数。
Java lambda表达式可以作为对象传递，并按需执行。
Java lambda表达式通常用于实现简单的事件侦听器/回调，或者使用Java Streams API进行函数式编程
###Java Lambdas和单一方法接口
函数式编程经常用于实现事件侦听器。Java中的事件侦听器通常使用单个方法定义为Java接口。这是一个假设的单一方法接口示例：
```java
public interface StateChangeListener {

    public void onStateChange(State oldState, State newState);

}
```
此Java接口定义了一个方法，只要状态发生变化（无论观察到什么），就会调用该方法。在Java 7中，您必须实现此接口才能监听状态更改。想象一下，你有一个名为StateOwner的类，它可以注册状态事件监听器。这是一个例子：
```java
public class StateOwner {

    public void addStateListener(StateChangeListener listener) { ... }

}
```
在Java7可以使用一个匿名内部接口实现类添加监听。就像下面
```java
StateOwner stateOwner = new StateOwner();

stateOwner.addStateListener(new StateChangeListener() {

    public void onStateChange(State oldState, State newState) {
        // do something with the old and new state.
    }
});
```
首先创建一个StateOwner实例。然后在StateOwner实例上添加StateChangeListener接口的匿名实现作为侦听器。在Java 8中，您可以使用Java lambda表达式添加事件侦听器，如下所示：
```java
StateOwner stateOwner = new StateOwner();

stateOwner.addStateListener(
    (oldState, newState) -> System.out.println("State changed")
);
```
lambda表达式是这一部分：
```java
(oldState, newState) -> System.out.println("State changed")
```
lambda表达式与addStateListener（）方法的参数的参数类型匹配。如果lambda表达式与参数类型（在本例中为StateChangeListener接口）匹配，则lambda表达式将转换为实现与该参数相同的接口的函数。Java lambda表达式只能在与它们匹配的类型是单个方法接口的情况下使用。在上面的示例中，lambda表达式用作参数，其中参数类型是StateChangeListener接口。该接口只有一个方法。因此，lambda表达式与该接口成功匹配。
###具有默认方法与静态方法的接口
从Java 8开始，Java接口可以包含默认方法和静态方法。默认方法和静态方法都有直接在接口声明中定义的实现。这意味着，Java lambda表达式可以实现具有多个方法的接口 - 只要接口只有一个未实现的（AKA抽象）方法。换句话说，只要接口只包含一个未实现的（抽象）方法，接口仍然是一个功能接口，即使它包含默认和静态方法。
可以使用lambda表达式实现以下接口：
```java
import java.io.IOException;
import java.io.OutputStream;

public interface MyInterface {

    void printIt(String text);

    default public void printUtf8To(String text, OutputStream outputStream){
        try {
            outputStream.write(text.getBytes("UTF-8"));
        } catch (IOException e) {
            throw new RuntimeException("Error writing String as UTF-8 to OutputStream", e);
        }
    }

    static void printItToSystemOut(String text){
        System.out.println(text);
    }
}
```
尽管此接口包含3个方法，但它可以通过lambda表达式实现，因为只有一个方法未实现。以下是实现的外观：
```java
MyInterface myInterface = (String text) -> {
    System.out.print(text);
};
```
###Lambda表达式与匿名接口实现
尽管lambda表达式接近于匿名接口实现，但仍有一些值得注意的差异。主要区别在于，匿名接口实现可以具有状态（成员变量），而lambda表达式则不能。看看这个接口：
```java
public interface MyEventConsumer {

    public void consume(Object event);

}
```

这个接口可以通过一个匿名内部类来实现，比如：
```java

MyEventConsumer consumer = new MyEventConsumer() {
    public void consume(Object event){
        System.out.println(event.toString() + " consumed");
    }
};
```
这个匿名内部类MyEventConsumer可以拥有属于自己的局部变量。看一下下面的这个重新设计：
```java

MyEventConsumer myEventConsumer = new MyEventConsumer() {
    private int eventCount = 0;
    public void consume(Object event) {
        System.out.println(event.toString() + " consumed " + this.eventCount++ + " times.");
    }
};
```
请注意匿名MyEventConsumer实现现在具有名为eventCount的字段。lambda表达式不能包含这样的字段。因此，lambda表达式被认为是无状态的。
###Lambda类型推断
在Java 8之前，你必须在进行匿名接口实现时指定要实现的接口。以下是本文开头的匿名接口实现示例：
```java
stateOwner.addStateListener(new StateChangeListener() {

    public void onStateChange(State oldState, State newState) {
        // do something with the old and new state.
    }
});
```
对于lambda表达式，通常可以从周遭的代码推断出类型。例如，参数的接口类型可以从addStateListener（）方法的方法声明（StateChangeListener接口上的单个方法）推断出来。这称为类型推断。编译器通过查找类型的其他地方来推断参数的类型 - 在本例中是方法定义。以下是本文开头的示例，显示lambda表达式中未提及StateChangeListener接口：
```java
stateOwner.addStateListener(
    (oldState, newState) -> System.out.println("State changed")
);
```
在lambda表达式中，通常也可以推断出参数类型。在上面的示例中，编译器可以从onStateChange（）方法声明中推断出它们的类型。因此，参数oldState和newState的类型是从onStateChange（）方法的方法声明中推断出来的。
###Lambda参数
由于Java lambda表达式实际上只是方法，因此lambda表达式可以像方法一样获取参数。前面显示的lambda表达式的（oldState，newState）部分指定了lambda表达式所采用的参数。这些参数必须与单个方法接口上的方法参数匹配。在这种情况下，这些参数必须匹配StateChangeListener接口的onStateChange（）方法的参数：
```java
public void onStateChange(State oldState, State newState);
```
lambda表达式和方法中的参数数量必须至少匹配。其次，如果在lambda表达式中指定了任何参数类型，则这些类型也必须匹配。我还没有向您展示如何将类型放在lambda表达式参数上（本文稍后将对其进行介绍），但在许多情况下您不需要它们。
###Lambda方法体
lambda表达式的主体，以及它所代表的函数/方法的主体，被指定在lambda声明中 - >的右侧：这是一个例子：
```java
(oldState, newState) -> System.out.println("State changed")
```
如果你的lambda表达式需要由多行组成，你可以将lambda函数体括在{}括号中，Java在其他地方声明方法时也需要这样。这是一个例子：
```java
 (oldState, newState) -> {
    System.out.println("Old state: " + oldState);
    System.out.println("New state: " + newState);
  }
```
###Lambda表达式返回值
您可以从Java lambda表达式返回值，就像从普通方法中一样。你只需向lambda函数体添加一个return语句，如下所示：
```java
 (param) -> {
    System.out.println("param: " + param);
    return "return value";
  }
```
如果你的lambda表达式所做的只是计算返回值并返回它，你可以用更短的方式指定返回值。而不是这个：
```java
 (a1, a2) -> { return a1 > a2; }
 ```
 你可以这样写：
 ```java
 (a1, a2) -> a1 > a2;
 ```
然后编译器会发现表达式a1> a2是lambda表达式的返回值（因此名称为lambda表达式 - 因为表达式返回某种值）。
###Lambda作为对象
Java lambda表达式本质上是一个对象。您可以将lambda表达式赋给变量并传递它，就像使用任何其他对象一样。这是一个例子：
```java
public interface MyComparator {

    public boolean compare(int a1, int a2);

}
MyComparator myComparator = (a1, a2) -> return a1 > a2;

boolean result = myComparator.compare(2, 5);
```
第一个代码块显示了lambda表达式实现的接口。第二个代码块显示了lambda表达式的定义，lambda表达式如何赋给变量，以及最后如何通过调用它实现的接口方法来调用lambda表达式。

###变量访问
Java lambda表达式能够访问在某些情况下在lambda函数体外声明的变量。
Java lambdas可以访问以下类型的变量
- 局部变量
- 实例变量
- 静态变量
####访问局部变量
Java lambda可以捕获在lambda体外声明的局部变量的值。为了说明这一点，首先看一下这个单一的方法接口：
```java
public interface MyFactory {
    public String create(char[] chars);
}
```
现在，看看这个实现MyFactory接口的lambda表达式：
```java
MyFactory myFactory = (chars) -> {
    return new String(chars);
};
```
现在这个lambda表达式只引用传递给它的参数值（字符）。但我们可以改变这一点。这是一个更新版本，引用在lambda函数体外声明的String变量：
```java
String myString = "Test";

MyFactory myFactory = (chars) -> {
    return myString + ":" + new String(chars);
};
```
如您所见，lambda体现在引用在lambda体外声明的局部变量myString。当且仅当引用的变量是“有效最终”时，这是可能的，这意味着它在分配后不会改变其值。如果myString变量稍后更改了它的值，编译器会抱怨lambda函数内部对它的引用。
####访问实例变量
lambda表达式还可以访问创建lambda的对象中的实例变量。这里有一个示例：
```java
public class EventConsumerImpl {

    private String name = "MyConsumer";

    public void attach(MyEventProducer eventProducer){
        eventProducer.listen(e -> {
            System.out.println(this.name);
        });
    }
}
```



