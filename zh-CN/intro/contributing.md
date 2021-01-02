---
name: 为 beego 贡献 sort: 1
---

# 为 beego 做贡献

## 简介

beego 是免费、开源的软件，这意味着任何人都可以为其开发和进步贡献力量。beego 源代码目前托管在 Github 上，Github 提供非常容易的途径 fork 项目和合并你的贡献。

## Pull Requests

pull request 的处理过程对于新特性和 bug 是不一样的。在你发起一个新特性的 pull request 之前，你应该先创建一个带有 `[Proposal]` 标题的 issue。这个proposal
应当描述这个新特性，以及实现方法。提议将会被审查，有可能会被采纳，也有可能会被拒绝。当一个提议被采纳，将会创建一个实现新特性的 pull request。没有遵循上述指南的 pull request 将会被立即关闭。

为 bug 创建的 Pull requests 不需要创建建议 issue。如果你有解决 bug 的办法，请详细描述你的解决方案。

对文档的增加和修改也可以提交到 Github 上的[文档源码库](https://github.com/beego/beedoc)。

提交新特性

如果你希望 beego 中出现某个新特性，你可以在 Github 中创建一个带有 `[Request]` 标题的 issue。该建议将会被核心代码贡献者审查。

现在 beego 采用的 gitflow 的开发方式，所以现有的所有开发都是在 develop 分支上面，因此所有的 pull request 请发到 develop 分支上。

## Git命令简介

准备工作: 如果你没有github账号, 您需要申请一个Github账号, 接下来可以继续下一步.

### Fork 代码

1. 访问 [https://github.com/beego/beego](https://github.com/beego/beego)
2. 点击 "Fork" 按钮 (位于页面的右上方)

### Clone 代码

一般我们推荐将`origin`设置为官方的仓库，而设置一个自己的`upstream`。

如果已经在`github`上开启了SSH，那么我们推荐使用SSH，否则使用HTTPS。两者之间的区别在于，使用HTTPS每次推代码到远程库的时候，都需要输入身份验证信息。

使用SSH：

```bash
git clone git@github.com:astaxie/beego.git
cd beego
git remote add upstream 'git@github.com:<your github username>/beego.git'
```

使用HTTPS：

```bash
git clone https://github.com/beego/beego.git
cd beego
git remote add  'https://github.com/<you github username>/beego.git'
```

`upstream`可以替换为任何你喜欢的名字。比如说你的用户名，你的昵称，或者直接使用`me`。后面的命令也要执行相应的替换。

### 同步代码

除非刚刚把代码拉到本地，否则我们需要先同步一下远程仓库的代码。

```bash
git fetch
```

在不指定远程库的时候，这个指令只会同步`origin`的代码。如果我们需要同步自己`fork`出来的，可以加上远程库名字：

```bash
git fetch upstream
```

### 创建 feature 分支

我们在创建新的 feature 分支的时候，要先考虑清楚，从哪个分支切出来。

我们假设，现在我们希望添加的特性将会被合并到`develop`分支，或者说我们的新特性要在`develop`的基础上进行，执行：

```bash
git checkout -b feature/my-feature origin/develop
```

这样我们就切出来一个分支了。该分支的代码和`origin/develop`上的完全一致。

### 提交 commit

```bash
git add .
git commit
git push upstream my-feature
```

### 提交 PR

```bash
访问 https://github.com/beego/beego/v2, 

点击 "Compare" 比较变更并点击 "Pull request" 提交 PR
```
