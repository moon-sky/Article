Gerrit Topic功能解决关联提交导致Jenkins构建失败问题

#### 问题背景：

本公司使用gerrit用于codereview，提交代码需要先提交到gerrit。本人总共进行了三次提交

Commit1:修改了xxxengine.java类

Commit2:同样修改了xxxxengine.java

Commit3：是对于commit2的补充提交，所以我使用了git commit --amend，然后push到gerrit

由于都是在一个分支做的修改，commit1与commit2建立了relation，所以在commit2的两个patch提交时，由于commit1还没有被合入远程git仓库，gerrit自动触发了Jenkins的自动构建，

![](https://gitee.com/moonsky/image-bed/raw/master/image-20210717113520969.png)

但是由于commit1的代码并未合入,导致后来的依赖commit1的提交构建失败

#### 解决方法：

1. 在gerrit界面给每个relation commit 增加topic![](https://gitee.com/moonsky/image-bed/raw/master/image-20211115095110837.png)
2. 每次提交代码之前，先checkout 一个新分支，然后review的时候选择对应的要合入的分支，他会自动增加添加topic

