title: github如何贡献自己的代码
date: 2015-12-06 20:39:07
tags: [github,git]
categories: 技巧
---

Github不用多说，基本都用过，但常用的就几个命令，`git add .` , `git commit -m` , `git push`神马的，如果说单机开发，或许足够了。不过要为某一项目共享自己力量，请遵循如下步奏：

首先fork开源项目

* 把fork过去的项目也就是你的项目clone到你的本地
* 在命令行运行`git branch develop`来创建一个新分支
* 运行`git checkout develop`来切换到新分支
* 运行`git remote add upstream`远端地址 把开源库添加为远端库
* 运行`git remote update`更新
* 运行`git fetch upstream gh-pages(你的工作分支)`拉取远端库的更新到本地
* 运行`git rebase upstream/gh-pages(你的工作分支)`将远端更新合并到你的分支
* 这是一个初始化流程，只需要做一遍就行，之后请一直在develop分支进行修改。

如果修改过程中源库有了更新，请重复6、7、8步。

修改之后，首先push到你的库，然后登录GitHub，在你的库的首页可以看到一个pull request 按钮，点击它，填写一些说明信息，然后提交即可。
