---
title: git-flow的分支模型
date: 2019-11-13 14:35:52
tags:
- Show All
- git-flow
header-img: "Demo.png"
---

##### git的基础
[git的一些命令](https://lzkgithub134679.github.io/2019/11/13/gitMin/)

##### git分支
在git上开分支是非常容易非常快速的事情，只需要创建一个引用指向当前的commit即可

##### git-flow
git flow 是2010年 一个叫nvie的外国人设计的开发模型，旨在保持git仓库的优雅并且整洁。
[A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/ "A successful Git branching model")

##### git-flow 初见
网上广为流传的，出自上文链接中的图片：
![](5dbb8bfb478cf.png)

##### git-flow详解
由于对git-flow的模型没有深刻的认知和实践，暂不记录内容。

##### 公司开发的分支模型
对git-flow的原模型进行借鉴，思考适合公司的开发模型。比如，提测后需求依然经常变更。
- master分支：init后就有，每次生产环境部署都是从master部署。（不打tag了）
- dev分支：init后分自master，一般不在dev上直接开发功能，经常作为feature分支与feature分支之间的沟通
- feature分支：分自dev，每开发一个新功能，就开辟一个新的feature分支，并且在feature分支上进行新功能的开发。期间会频繁的将dev的代码merge到自己的feature，防止将来并回dev时冲突太多。
- release分支：分自dev，提测的时候，开辟一个新的release分支。改Bug在此分支上进行。测试通过后，并到master分支进行生产部署。每个release的提交节点都是具备可测试的。改Bug在release节点更新后，一般会同时merge到dev分支。

##### 备注1
1. feature分支如果认为功能开发完了，可以测试了，就merge到dev进行联调，并开辟release分支测试；
2. feature会频繁的将dev的代码merge到自己，防止将来合并回dev时冲突太多；
3. release修改Bug后一般会同时merge到dev，但是绝不能再从其他分支merge到release；
4. 如果'release-作业'正在测试，'feature-任务'开发完成需要提测，则'feature-任务'merge到dev，并开辟新的'release-作业任务'进行提测，'release-作业'分支不再关注（可以删除了的意思）；
5. 公司目前的情况，可能会同时存在两个release分支，旧的对应生产环境的代码，用来更改生产环境的Bug。新的release是用来测试的，作用同上一条；
6. 公司目前的情况，提测后需求也是会改变的。即release分支已经生效并且在测试的过程中，需求变更了。
6.1 如果这个变更只需要涉及到自己，不需要跟其他人配合，那就等同于Bug来处理，直接在当前release上开发。（前提是认为这个开发时间短）
6.2 如果这个变更需要多人配合，那就当作feature开发，然后重新打release提测。
7. 在dev分支下联调。
8. dev分支比功能提测的release超前后，需要重新打release分支提测功能
9. 正常情况下，开发环境部署的就是dev分支，测试环境部署的是release，生产环境部署的是master

#### 备注2
一些git-flow的参考文档
- [干货：基于 Git Flow 的 Git 最佳实践（附加解决大家经常碰到的问题）](https://www.cnblogs.com/mark888/p/7222242.html "干货：基于 Git Flow 的 Git 最佳实践（附加解决大家经常碰到的问题）")
- [Git 在团队中的最佳实践--如何正确使用Git Flow](https://www.cnblogs.com/cnblogsfans/p/5075073.html "Git 在团队中的最佳实践--如何正确使用Git Flow")
- [git flow的使用](https://www.cnblogs.com/lcngu/p/5770288.html "git flow的使用")
- [介绍一个成功的 Git 分支模型](https://www.oschina.net/translate/a-successful-git-branching-model "介绍一个成功的 Git 分支模型")
- [基于git的源代码管理模型——git flow](https://www.ituring.com.cn/article/56870 "基于git的源代码管理模型——git flow")

#### 备注3
记录Q&A
1，Q: 如果碰到提测后需求有变更，后台接口升级不兼容，开feature分之后，后台开发完就可以并到dev吗？比如5人综合批的需求。
A: 不可以。 开feature开发完之后，讲道理是不是直接并到dev的，因为pad端开发很滞后，需要很久才能调通。而web端开发已经结束。那怎么联调呢？建议开发环境新开一台服务器，部署这个feature分支，不影响其他dev上原有接口的的代码联调。（这中间其实有一个被忽略的地方：Pad端其实尚未开发完成，不具备dev上联调的前提，所以不往dev合并）

#### 备注4
- 对于后端接口来说，如果开发的是新接口，或者接口升级是兼容的，在feature分支上开发完成后，为方便前端调用开发，为方便前端调用开发，为方便前端调用开发，可以并到dev分支让前端直接调用开发等待联调。