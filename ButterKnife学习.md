## Android 注入利器 Butterknife 实战篇 ##
###  前言  ###
之所以想去尝试注入这种方式去编写相关代码，还是因为客户端的涉及view的代码看起来太多了。而注入的方式，可以有效的减少这种代码数量。
### Butternife github 地址 ###
[http://jakewharton.github.io/butterknife/](http://jakewharton.github.io/butterknife/ "Butterknife 主页")
### 实战步骤 ###
1. 如果使用eclipse 开发 请下载最新[ButterKnife jar包](https://search.maven.org/remote_content?g=com.jakewharton&a=butterknife&v=LATEST)； 
如果是Android studio 你使用gradle的话请`compile 'com.jakewharton:butterknife:7.0.1'`如果是maven则
```<dependency>
  <groupId>com.jakewharton</groupId>
  <artifactId>butterknife</artifactId>
  <version>7.0.1</version>
</dependency>```由于本人习惯eclipse开发，下面就是按照eclipse的方式进行讲解了。
1. 将下载的最新的jar包放在lib目录下![](http://i.imgur.com/xiX3R7n.png)，这里需要特别注意的是，需要再eclipse里面设置一下，右键项目-->选择properties-->Java compiler-->Annotation processing 勾选如图所示的选项，![](http://i.imgur.com/XYb76DJ.png)接着再选择FactoryPath 添加刚才的butterknife的jar包![](http://i.imgur.com/Nt2J4vs.png)
2. 创建自己的Android project，编写布局这里就不细说了。接下来说一下如何使用butterknife
3. 声明一个View控件的方法之前是通过@inject 现在改为@Bind，例子：`	@Bind(R.id.button1)
	Button button;`
1. 注册点击事件 
	1. 方法一：```@OnClick(R.id.button1)
	void hello() {
		Toast.makeText(this, "hello", Toast.LENGTH_SHORT).show();
	}```
	1. 方法二：`@OnClick(R.id.button1)
	void hello(Button btn) {
		btn.setText("hello");
		Toast.makeText(this, "hello", Toast.LENGTH_SHORT).show();
	}`
