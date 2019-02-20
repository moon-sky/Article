## JAVA异常处理机制 ##
### 原则 ###
1. 调用者关心，则throw出去；不关心则try……catch 并且内部处理
	2. 有返回值的则throw，没有返回值的则try……catch  
2. 不让catch方法体为空
3. 尽早捕获
4. Use checked expections for recoverable conditions and runtime exceptions for programming errors (From Effective java)
5. 有些人的观点是不应该使用checked Exception ,因为这样会让你忽略掉很多重要的问题，降低代码的质量
6. 有些人认为 checked Exception 是必要的，这样会让方法的调用者处理异常这种情况
7. 对于什么时候 throw 什么时候方法内部处理，也没有统一的定论。建议：根据 该方法是否有返回值，返回值是否对于其调用者是否影响其逻辑。如果影响则throw。
8. 对于重要的异常，应该写在log日志里面方便测试人员查看，也方便开发人员后续跟进

### 类型 ###
1. checked 异常 系统检测到可能发生的异常 IOException
2. unChecked 异常 未被检测到的异常 例如 RuntimeException

### 参考资料 ###
1. [http://stackoverflow.com/questions/6115896/java-checked-vs-unchecked-exception-explanation](http://stackoverflow.com/questions/6115896/java-checked-vs-unchecked-exception-explanation "Java: checked vs unchecked exception explanation")
2. [http://stackoverflow.com/questions/499437/in-java-when-should-i-create-a-checked-exception-and-when-should-it-be-a-runti](http://stackoverflow.com/questions/499437/in-java-when-should-i-create-a-checked-exception-and-when-should-it-be-a-runti)
3. [http://stackoverflow.com/questions/27578/when-to-choose-checked-and-unchecked-exceptions](http://stackoverflow.com/questions/27578/when-to-choose-checked-and-unchecked-exceptions)