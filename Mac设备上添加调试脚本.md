### Mac设备上添加调试脚本

#### 编写脚本

在你平时存放脚本的地方，新建一个debug文件

将命令adb shell am start -a carspeechservice.action.debug 拷贝到文件中

![image-20210715142504511](https://gitee.com/moonsky/image-bed/raw/master/image-20210715142504511.png)

#### 更改执行权限

如果直接运行./debug

会提示权限不允许

```shell
zsh: permission denied: ./debug
```

执行命令

```shell
chmod +xu debug
```

然后再运行即可

#### 加入系统环境变量

如果你使用是bash shell，则再~/.bash_profile中添加debug文件路径

如果使用的是zsh shell，则在~/.zshrc中添加路径

#### 立即生效

运行source ~/.bash_profile 或者~./zshrc即可立即使用



#### 验证效果

![image-20210715143105622](https://gitee.com/moonsky/image-bed/raw/master/image-20210715143105622.png)