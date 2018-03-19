# git-flow实战

团队开发中，约定好一个统一的工作流程至关重要。git使用非常灵活，如果团队不形成一个特定有效的工作流程，那么就会非常混乱不堪。

Vincent Driessen发明了一套成功的git分支模型，下面我们先对该模型进行简单介绍。

## 分支模型
先上一张分支模型图:
![](img/git-model.png)

## git--分散但又集中
我们使用以及使用该分支模型工作良好的仓库配置是具有一个中心的事实仓库。 注意这个仓库只是被认为是中心仓库(因为git是分散版本控制系统, 在技术层面是没有这样的中心仓库的).
我们将这个仓库叫做origin, 这个名字有点类似git用户.

![](img/center-decent.png)

## 参考资料
1. [A successful git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
2. [git-flow备忘清单](https://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
