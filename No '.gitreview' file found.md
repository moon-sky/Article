## No '.gitreview' file found

[TOC]



### 问题背景

从公司gerrit上拉下来一个代码，修改了一些东西，按照流程应该使用git review +分支名

可是出现了错误：

![image-20211117182458274](https://gitee.com/moonsky/image-bed/raw/master/image-20211117182458274.png)



### 解决办法

#### 方法1

手动创建.gitreview文件

格式参考：

```shell
[gerrit]
host=ci.test.local//gerrit地址
port=29418//gerrit端口号
project=Android_Common/ApkDependency.git //仓库名称加.git
defaultbranch=develop
```

自己验证没问题

#### 方法2

```shell
git remote rename origin gerrit
```

未验证

