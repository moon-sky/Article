### Typora+Gitee+PicGo实现markdown图片自动插入

[TOC]



#### 写到前面

现在用markdown编写技术文章已经成为一个主流,而typora拼接它免费、实时预览等特色功能也称为mac用户的最爱.

在我们编写技术文章的时候,相较于只是堆砌单纯的文字+代码,大家更喜欢图文并貌的风格.在插入图片的时候,很多时候会使用本地的图片或者是截图,那么markdown引用的图片路径也是本地路径,当我们要将markdown的文章上传到技术博客或者是内部wiki(例如confluence)的时候,就会出现图片找不到的问题.

为了解决这个问题,大部分人会将图片上传到免费的图床网站,然后把链接复制到md中,如果只是一张两张图片还可以忍受,但是如果图片数目较大的时候,就会很浪费时间.

而本篇文章要介绍的就是这样一个自动将图片上传至自建图床的方案.



#### 效果展示

<img src="https://gitee.com/moonsky/image-bed/raw/master/image-20210709135549246.png" alt="image-20210709135549246" style="zoom:50%;" />

这个图是我用微信自带的截图工具截的图,ctrl+v粘贴进md中,它会自动上传至

gitee仓库中![image-20210709142751568](https://gitee.com/moonsky/image-bed/raw/master/image-20210709142751568.png)



#### 搭建步骤

##### 下载并安装Picgo

[Picgo官网](https://picgo.github.io/PicGo-Doc/zh/guide/#%E7%89%B9%E8%89%B2%E5%8A%9F%E8%83%BD)

里面有各个操作系统的安装方式,我这里只是说一下mac的安装方式.大家如果用mac一定知道homebrew这个软件的赫赫大名，如果还没有安装的，请自行搜索安装，因为接下来的node.js也需要使用。

安装picgo,一行命令足够

```shell
brew cask install picgo
```

等它安装就可以。

等你看到以下日志，就说明安装成功了

```sh
==> Moving App 'PicGo.app' to '/Applications/PicGo.app'
🍺  picgo was successfully installed!
```

接下来你就可以回到桌面，点击picgo图标,一般会提示,软件不可信任之类的.

进入‘系统偏好设置’->'安全性与隐私' 选择允许picgo运行即可.

##### 安装node.js

因为我们需要安装picgo中gitee插件,而这些插件有需要node.js环境,所以在选择插件之前,请先按照node.js

安装方法同样很简单:

```sh
brew install node
```

安装成功以后，就可以安装gitee插件了

##### 创建gitee仓库

大家访问https://gitee.com/官网，按照步骤注册账户即可。

注册成功后，点击右上角的➕号，选择新建仓库

这里要注意的是仓库一定要设置为**公开的**，否则无法使用。

![image-20210709150705461](https://gitee.com/moonsky/image-bed/raw/master/image-20210709150705461.png)

创建成功以后，我们就可以看到对应的仓库了。



##### 安装gitee插件

打开picgo主界面，选择‘插件设置’tab，输入gitee，选择箭头所示的这个，点击安装，安装后，重启以下picgo，



![image-20210709145901735](https://gitee.com/moonsky/image-bed/raw/master/image-20210709145901735.png)

在左侧图床设置中就会看到gitee的身影了。

点击gitee，就可看到gitee设置界面，这里需要的数据，我会挨个说明。

![image-20210709150310107](https://gitee.com/moonsky/image-bed/raw/master/image-20210709150310107.png)

第一个：repo是仓库的路径，这里以仓库的url的path为准，而不是仓库名称，这里要特别注意。如下图所示，红色标注的才是repo的真正路径

![image-20210709151040724](https://gitee.com/moonsky/image-bed/raw/master/image-20210709151040724.png)

第二个：branch分支，一般默认就是master，没有必要不建议更改

第三个：token是用户私有令牌，它的获取方式是，进入gitee官网，进入![image-20210709151334662](https://gitee.com/moonsky/image-bed/raw/master/image-20210709151334662.png)

接下来选择私人令牌，新建私人令牌，并复制一下，粘贴到picgo-gitee插件中的token，最后点击确认就可以了。

![image-20210709152704857](https://gitee.com/moonsky/image-bed/raw/master/image-20210709152704857.png)

##### 设置Typroa

打开Typroa->偏好设置->上传服务选择PicGo.app即可

![image-20210709170752688](https://gitee.com/moonsky/image-bed/raw/master/image-20210709170752688.png)

#### 总结

工欲善其事必先利其器，希望能够帮到大家.