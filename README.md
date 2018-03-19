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

#### 创建一个发布分支
发布分支从develop分支创建。例如，假设1.1.5是当前的产品版本，同时我们即将发布下个版本。develop分支的状态已经是准备好“下一版本”发布了，我们也决定下个版本是1.2（而不是1.1.6或者2.0）。因此我们创建发布分支，并且为其赋予一个能体现新版本号的名称：

```
$ git checkout -b releases-1.2 develop
Switched to a new branch “release-1.2”
$ ./bump-version.sh 1.2
Files modified successfully. version bumped to 1.2.
$ git commit -a -m “Bumped version number to 1.2”
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed. 1 insertions(+). 1 deletions(-)
```

#### 结束一个发布分支
当发布分支达到一个可以正式发布的状态时，我们就需要执行一些操作。首先，将发布分支合并至master（记住，我们之前定义master分支上的每一个commit都对应一个新版本）。接着，master分支上的commit必须被打上标签（tag），以方便将来寻找历史版本。最后，发布分支上的变更需要合并回develop，这样将来的版本也能包含相关的bug修复。

前两步在Git中的操作：


```
$ git checkout master
Switched to branch ‘master’
$ git merge –no-ff release-1.2
Merge made by recursive.
(Summary of changes)
    $ git tag -a 1.2
```

现在版本发布完成了，而且为未来的查阅提供了标签。

提醒：你可能同时也会想要用 -s 或者 -u <key> 来对标签进行签名。

为了能保留发布分支上的变更，我们还需要将分支合并回develop。在Git中：
```
$ git checkout develop
Switched to branch ‘develop’
$ git merge –no-ff release-1.2
Merge made by recursive.
(Summary of changes)
```

这一操作可能会导致合并冲突（可能性还很大，因为我们改变了版本号）。如果发现，则修复之并提交。

现在工作才算真正完成了，最后一步是删除发布分支，因为我们已不再需要它：

```
$ git branch -d release-1.2
Deleted branch release-1.2 (was ff452fe).
```

### 热补丁分支
可能的分支来源：master
必须合并回：develop和master
分支命名约定：hotfix-*

热补丁分支和发布分支十分类似，它的目的也是发布一个新的产品版本，尽管是不在计划中的版本发布。当产品版本发现未预期的问题的时候，就需要理解着手处理，这个时候就要用到热补丁分支。当产品版本的重大bug需要立即解决的时候，我们从对应版本的标签创建出一个热补丁分支。

使用热补丁分支的主要作用是（develop分支上的）团队成员可以继续工作，而另外的人可以在热补丁分支上进行快速的产品bug修复。

#### 创建一个热补丁分支
热补丁分支从master分支创建。例如，假设1.2是当前正在被使用的产品版本，由于一个严重的bug，产品引起了很多问题。同时，develop分支还处于不稳定状态，无法发布新的版本。这时我们可以创建一个热补丁分支，并在该分支上修复问题：
```
$ git checkout -b hotfix-1.2.1 master
Switched to a new branch “hotfix-1.2.1″
$ ./bump-version.sh 1.2.1
Files modified successfully, version bumped to 1.2.1.
$ git commit -a -m “Bumped version number to 1.2.1″
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)
```

不要忘了在创建热补丁分之后设定一个新的版本号！

然后，修复bug并使用一个或者多个单独的commit提交。

```
$ git commit -m “Fixed severe production problem”
[hotfix-1.2.1 abbe5d6] Fixed severe production problem
5 files changed, 32 insertions(+), 17 deletions(-)
```

#### 结束一个热补丁分支
修复完成后，热补丁分支需要合并回master，但同时它还需要被合并回develop，这样相关的修复代码才会同时被包含在下个版本中。这与我们完成发布分支很类似。

首先，更新master分支并打上标签。
```
$ git checkout master
Switched to branch ‘master’
$ git merge –no-ff hotfix-1.2.1
Merge made by recursive.
(Summary of changes)
$ git tag -a 1.2.1
```
> 提醒：你可能同时也会想要用 -s 或者 -u <key> 来对标签进行签名。

接着，将修复代码合并到develop：
```
    $ git checkout develop
    Switched to branch ‘develop’
    $ git merge –no-ff hotfix-1.2.1
    Merge made by recursive.
    (Summary of changes)
```
这里还有个例外情况，如果这个时候有发布分支存在，热补丁分支的变更则应该合并至发布分支，而不是develop。将热补丁合并到发布分支，也意味着当发布分支结束的时候，变更最终会被合并到develop。（如果develop上的开发工作急需热补丁并无法等待发布分支完成，这时你也已经可以安全地将热补丁合并到develop分支。）

最后，删除临时的热补丁分支：
```
$ git branch -d hotfix-1.2.1
Deleted branch hotfix-1.2.1 (was abbe5d6).
```


虽然这个分支模型中没有什么特别新鲜的东西，但本文起始处的“全景图”事实上在我们的项目中起到了非常大的作用。它帮助建立了优雅的，易理解的概念模型，使得团队成员能够快速建立并理解一个公用的分支和发布过程。


## git-flow工具

## 参考资料
1. [A successful git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
2. [git-flow备忘清单](https://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
