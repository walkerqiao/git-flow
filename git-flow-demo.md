# git-flow demo

## 初始化
```
git flow init
```

这一步执行一大推的询问，建议使用默认值，一直下去。

## 特性

### 开发新特性

```
git flow feature start myfeature
```

基于develop分支创建一个特性分支，并切换到这个特性分支下面。

### 完成新特性

```
git flow feature finish myfeature
```

完成新特性。这个动作执行下面的操作:
1. 合并myfeature分支到develop分支
2. 删除myfeature分支
3. 切回到develop分支

### 发布新特性
我们日常项目开发中，通常都是合作开发一些新特性，这个时候，我们建立的新特性希望其他合作者能够从远端服务器上拉取到，那么我们就需要将该新特性分支发布到远程服务器。

```
git flow feature publish myfeature
```

执行该命令后，远程仓库中就有一个feature/myfeature分支了。

### 取得一个发布的新特性分支

取得其它用户发布的新特性分支，并签出远程的变更。
```
git flow feature pull origin myfeature
```
也可以使用
```
git flow feature track MYFEATURE
```

跟踪在origin上的特性分支。

## 做一个release版本

开始准备release版本，使用git flow release命令。

它从develop分支开始创建一个release分支。

```
git flow release start RELEASE [BASE]
```

这里RELEASE是要创建的release的名称，另外可以提供一个[BASE]参数，即提交记录的sha-1 hash值，来开启动release分支，这个提交记录的sha-1 hash值必须是develop分支下的。

创建 release 分支之后立即发布允许其它用户向这个 release 分支提交内容是个明智的做法。命令十分类似发布新特性：

```
git flow release publish RELEASE
```

你可以通过

```
git flow release track RELEASE
```
命令签出 release 版本的远程变更)


### 完成release版本

完成 release 版本是一个大 git 分支操作。它执行下面几个动作：

* 归并release分支到'master'分支
* 用release分支名打Tag
* 归并release分支到'develop'
* 移除release分支

```
git flow release finish RELEASE
```

## 紧急修复

* 紧急修复来自这样的需求：生产环境的版本处于一个不预期状态，需要立即修正。
* 有可能是需要修正master分支上某个TAG标记的生产版本。

和其他git-flow命令一样，紧急修复分支开始于:
```
git flow hotfix start VERSION [BASENAME]
```
## 参考链接

* [git-flow命令备忘](https://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
```
