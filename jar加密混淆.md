# jar 加密 #

首先看一下stackoverflow上面大家的讨论

![](http://i.imgur.com/DVYAcVO.png)

jar包本身加密是不可能的，这是由JVM的机制决定的。JVM肯定会知道它要执行的代码内容，而只要通过合适的工具，我们就可以获取到JAR包的内容。我们只剩下一条路--混淆代码。
常见的反编译方案

1. dex2jar（获取源码class jar） + jdgui（源码浏览）+apktool（资源文件获取）

常见的混淆方案：

1. [ Jshrink](http://www.e-t.com/jshrink.html) 号称最好的代码保护工具，提供命令行和GUI界面
2. [Proguard](https://www.guardsquare.com/en/proguard) 大名鼎鼎的proguard是很多IDE默认的代码混淆工具。如果你不想用命令行，它也提供一个简单化的GUI界面

