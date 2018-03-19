# git-flow实战

团队开发中，约定好一个统一的工作流程至关重要。git使用非常灵活，如果团队不形成一个特定有效的工作流程，那么就会非常混乱不堪。

Vincent Driessen发明了一套成功的git分支模型，下面我们先对该模型进行简单介绍。

## 分支模型
先上一张分支模型图:
![](img/git-model.png)

## git--去中心化但又保持中心化

![](img/center-decent.png)

git仓库是DVCS(分布式版本控制系统), 在技术层面上是没有中心这个概念的。我们后面将origin当作这个中心，每个开发者都从origin做push和pull操作。不过除了这种中心化的push-pull关系之外， 每个开发者还可以从其他开发者或小组处pull变更。 例如，在大型项目中，有多个开发者共同开发一个大特性，在往origin永久性的push工作代码之前，他们之间可以执行一些去中心化的操作。 例如上图所示，alice和bob, alice和david, clair和david他们可以组成小组。
从技术上来说，这仅仅是Alice定一个git remote, 名字为bob, 指向bog的仓库，反过来也一样。

## 主要分支
此开发模式受现有开发模式影响。中心仓库包含了两个主要分支，这两个分支是在整个生命周期存活的。

* master
* develop

origin/develop分支上的HEAD反映了开发过程中最新的提交变更。 有人称之为“集成分支”。 该分支为自动化每日构建的代码源。

当develop分支上的源码到达一个稳定的状态，就可以发布版本。 所有develop上的变更都应以某种方式合并回master分支，并且使用发布版本打上标签。

印次，每次有变化被合并到master分支时，根据定义这就是一次新的产品版本发布。我们趋向于严格遵守该规范，所以理论上来说，每次master有提交时，我们都可以使用一个git钩子(hook)脚本来自动构建并部署软件至产品生产环境。

## 支持性分支
紧接着主要分支master和develop，我们的开发模型使用多种支持性分支来帮助团队成员间实现并行开发、追踪产品特性、准备产品版本发布、以及快速修复产品问题。与主要分支不同的是，这些分支的寿命是有限的，它们最终都会被删除。

我们会用到的分支有这几类：

* 特性分支（feature branch）
* 发布分支（release branch）
* 热补丁分支（hotfix branch）

上述每种分支都有特定的用途，它们各自关于源自什么分支、合并回什么分支，都有严格的规定。稍后我们逐个进行介绍。

从技术角度来说，这些分支一点都不“特殊”。分支按照我们对其的使用方式进行分类。技术角度它们都一样是平常的Git分支。

### feature分支
可能的来源分支: develop
必须合并回的分支: develop
分支命名约定: 除了使用master, develop, release-xx, hotfix-xx之外的名字都可以。

特性分支(有时候叫做话题分支topic branches)用于为下一个发布版本开发新特性。

当开始开发一个特性的时候，该特性会成为哪个发布版本的一部分，往往还不知道。特性分支的重点是，只要特性还在开发，该分支就会一直存在，不过它最终会被合并回develop分支（将该特性加入到发布版本中），或者被丢弃（如果试验的结果令人失望）。
特性分支往往只存在于开发者的仓库中，而不会出现在origin。

#### 创建特性分支
```git checkout -b myfeature develop```

#### 合并完成的特性回develop
完成的特性应该被合并回develop分支以将特性加入到下一个发布版本中：
```
git checkout develop

git merge -no-ff myfeature

git branch -d myfeature

git push origin develop
```


上述代码中的–no-ff标记会使合并永远创建一个新的commit对象，即使该合并能以fast-forward的方式进行。这么做可以避免丢失特性分支存在的历史信息，同时也能清晰的展现一组commit一起构成一个特性。比较下面的图：
![](img/merge-without-ff.png)

在第2张图中，已经无法一眼从Git历史中看到哪些commit对象构成了一个特性——你需要阅读日志以获得该信息。在这种情况下，回退（revert）整个特性（一组commit）就会比较麻烦，而如果使用了–no-diff就会简单很多。

是的，这么做会造成一些（空的）commit对象，但这么做是利大于弊的。

可惜的是，我没能找到方法让–no-diff成为默认的git merge行为参数，但其实应该这么做。

### 发布分支release branch
可能的分支来源：develop
必须合并回：develop和master
分支命名约定：release-*

发布分支为准备新的产品版本发布做支持。它允许你在最后时刻检查所有的细节。此外，它还允许你修复小bug以及准备版本发布的元数据（例如版本号，构建日期等等）。在发布分支做这些事情之后，develop分支就会显得比较干净，也方便为下一大版本发布接受特性。

从develop分支创建发布分支的时间通常是develop分支（差不多）能反映新版本所期望状态的时候。至少说，这是时候版本发布所计划的特性都已经合并回了develop分支。而未来其它版本发布计划的特性则不应该合并，它们必须等到当前的版本分支创建好之后才能合并。

正是在发布分支创建的时候，对应的版本发布才获得一个版本号——不能更早。在该时刻之前，develop分支反映的是“下一版本”的相关变更，但不知道这“下一版本”到底会成为0.3还是1.0，直到发布分支被创建。版本号是在发布分支创建时，基于项目版本号规则确定的。

## 参考资料
1. [A successful git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
2. [git-flow备忘清单](https://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
