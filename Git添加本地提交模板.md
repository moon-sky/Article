### Git添加本地提交模板

很多公司对于git提交有一定的规范，如果可以设置git提交模板，将会提高这种规范性，并且减少手动敲打commit信息的无效功。接下来，咱们就开始手把手设置本地模板吧。

#### 编写模板

新建一个文件，可以命名为commitTemplate（当然了，看个人喜好，这里不重要）

打开之后，将你们公司要求的提交模板写进去,我把我们公司的给大家看一下

```shell
[ Modify      ] 
[ Project     ] 
[ Module      ] 
[ Description ] 
[ Reference   ] 
[ Company     ] 
[ Team        ] 

# Type 可以参考以下，自行添加:
# * Feature (功能相关)
# * Bugfix (修复bug)
# * Docs (修改文档)
# * Style (代码排版)
# * Refactor (重构)
# * Test (测试代码)
# * Modify ()
# * Merge (合入官方补丁)
# Project 是指本次提交影响到的项目，比如xxxx
# Module 是指业务上修改的模块，比如xxx
# Description 详细描述, 比如打google补丁时，可以把原来的commit message填在这里
# Reference jira详细链接

```

大家可以自行修改

#### 设置模板

执行命令

```shell
git config --global commit.template path(文件路径)
```

设置完成，接下来只需要在提交修改代码的时候，

输入git commit即可看到模板，是不是很简单，有时候简单的事情反而能提高效率，如果大家有更多的建议，请评论留言

