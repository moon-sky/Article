### DLNA ###
#### 概念 ####
DLNA的全称是DIGITAL LIVING NETWORK ALLIANCE(数字生活网络联盟)， 其宗旨是Enjoy your music, photos and videos, anywhere anytime， DLNA(Digital Living Network Alliance) 由索尼、英特尔、微软等发起成立、旨在解决个人PC，消费电器，移动设备在内的无线网络和有线网络的互联互通，使得数字媒体和内容服务的无限制的共享和增长成为可能。
#### 应用场景 ###
1. 手机上的视频投射到电视上
![](http://p2.pstatp.com/large/2164/1982220824)
2. 终端之间文件互传（手机与手机，手机与电脑，手机与机器人（设想））
3. 手机直接连接打印机打印文件
![](http://www.45it.com/picture/allimg/141106/150Q325I-4.jpg)

#### 使用方式 ###
- 硬件要求：wifi环境,支持UPNP协议

#### 与机器人可能性的结合点 ####
1. 家长端与机器人互动
	1. 文件互传
	2. 家长端（获取当前机器人的硬件信息、软件信息等互动）
2. 工作与私人助理机器人
	1. 例如对机器人说 把我上次的日程传到我现在的电脑上
	2. 类似her中的文档修正，机器人帮我检查一下我这个文档中的错别字、语法错误
	3. 帮我处理一下这个图片，然后再传回来

3. 等等

#### 其他相关技术 ####
1. AirPlay (Apple公司)
2. MirCast
这两项技术更牛，可以实现屏幕镜像功能，类似于投影仪的功能，可以见这篇文章
[http://www.igao7.com/news/201406/airplay-dlna-miracast.html](http://www.igao7.com/news/201406/airplay-dlna-miracast.html)

#### 相关文章 ####
1. [http://www.techhive.com/article/2020825/how-to-get-started-with-dlna.html](http://www.techhive.com/article/2020825/how-to-get-started-with-dlna.html)
2. [https://www.zhihu.com/question/22927139](https://www.zhihu.com/question/22927139)
3. [http://www.45it.com/www/201411/38071.htm](http://www.45it.com/www/201411/38071.htm)


### UPNP 协议 ###
#### 概念 ####
英文名称：Universal Plug and Play
中文译名：通用即插即用

UPnP是由“通用即插即用论坛”（UPnP™ Forum）推广的一套网络协议。该协议的目标是使家庭网络（数据共享、通信和娱乐）和公司网络中的各种设备能够相互无缝连接，并简化相关网络的实现。UPnP通过定义和发布基于开放、因特网通讯网协议标准的UPnP设备控制协议来实现这一目标。
UPnP体系允许 PC 间的点对点连接、网际互连和无线设备。它是一种基于TCP/IP、UDP和HTTP的分布式、开放体系。
#### 特点 ####
UPnP使得任意两个设备能在LAN控制设备的管理下相互通信。其特性包括：

1. 传输介质和设备独立。

	UPnP 技术可以应用在许多媒体上，包括电话线、电线（电力线通信PLC）、以太网、红外通信技术(IrDA)、无线电（Wi-Fi，蓝牙）和Firewire(1394)。无需任务设备驱动；而是采用共同的协议。

2. 用户界面（UI）控制。

  	UPnP 技术使得设备厂商可以通过网页浏览器来控制设备并进行交互。

3. 操作系统和程序语言独立。

	任何操作系统和程序语言均可以用于构建UPnP产品。UPnP 并没有设定或限制运行于控制设备上的应用程序API；OS厂商可以创建满足他们客户需求的API。UPnP使得厂商可以像开发常规应用程序一样来控制设备UI和交互。

4. 基于因特网技术。

	UPnP 构建于 IP, TCP, UDP, HTTP，和 XML 等许多协议之上。

5. 编程控制。
UPnP 体系同时支持常规应用程序编程控制。

6. 扩展性。
每个 UPnP 设备都可以有构建于基本体系之上、与具体设备相关的服务。

详情见链接[http://www.h3c.com.cn/MiniSite/Technology_Circle/Net_Reptile/The_Five/Home/Catalog/201206/747039_97665_0.htm](http://www.h3c.com.cn/MiniSite/Technology_Circle/Net_Reptile/The_Five/Home/Catalog/201206/747039_97665_0.htm)

UPnP 支持零配置，"看不见的网络" 和自动检测；任何设备能自动加入一个网络， 获取一个 IP 地址，宣布自己的名字，根据请求检查自身功能以及检测出其它设备 和它们的功能。DHCP 和 DNS 服务是可选的，并只有它们在网络上存在的时候才会 使用。设备可以自动离开网络而不会遗留下任何不需要的状态信息。