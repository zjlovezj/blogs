## 背景

git flow 作为一个最流行的的 git 协作指南，网上有很多教程，但是这些教程都过于简化。  
要么在某些地方语焉不详，造成错误理解，要么是没介绍常用的详细操作，造成使用的不便。  
我曾是其中困惑的一员，这次详细的把 git flow 中每个环节，每个角色都思考了一遍，写下了这篇文档，希望对大家有帮助。  
本文适合有一定 git 操作基础的人，并不适合做 git 入门。

## 流程演化

流程并不是天生的，也不是一开就固定写死的。关键要弄清楚流程背后的原因，知道原因以后，就明白为什么当前的流程是最适合的，也能在流程上偶尔做一些合理的小变动。

举例说明，项目刚开始的 git 流程和项目已经成熟以后的 git 流程需要一样吗？  
答案是不需要。  
因为项目刚开始的时候，都没有正式的测试和发布环节，并且一开始提交的东西相互影响的非常多，所以流程设计上这和项目成熟期是非常不一样的。  
因为项目刚开始大家提交的基础代码比较多，需要时刻共享和更新，所以最好就在 master 上不断的交互提交。  
因为刚开始也不需要正式的测试和发布，所以也不用测试和发布的分支。  
偶尔有很不稳定的基础代码要编写时，此时也可以单开一个 feature 分支，等到稳定时再合并到 master。

而成熟期的项目就不一样，我们仅以固定发布周期的项目为例，因为存在一种持续集成持续发布的项目，这会很不一样，总之 git 的流程取决于项目的需求。  
成熟期的项目需求如下：

1. 有一个稳定的版本供线上发布
1. 有一个测试的分支供测试，修复 bug，并形成一个稳定的线上版本
1. 开发人员可以独立拥有分支，在这个分支上进行开发，后续再提交测试
1. 针对线上版本、测试分支、独立分支都有 bug 修复的操作流程，避免遗漏，并尽量简单
1. 针对线上版本、测试分支、独立分支都有代码回滚的操作流程，并保持代码历史清晰，回滚以后可以再恢复

## 分支详述

**特别注意** 仅有 develop 和 master 是长期分支，并且 master 后期是指向 develop 的某个 commit，所以 develop 分支是需要特别维护，如果有可能专人维护的话，提交历史会更清晰。  
**注意** 请注意分支的使用人员，某些分支仅供部分人员使用，某些分支是全体共享的。  
**注意** 请注意 commit 所处的发布周期，在正式发布前，各个分支的新 commit 都是可能被改写的，commit 被发布后，则严禁改写。  
**注意** 发布的版本号为 v{major}.{minor}.{fix}，当切出 release 和 bugfix 分支时，都应该跟上版本号。

### develop 分支

develop 是最常用的共享分支。它在项目 feature 开发基本完成，准备进入测试阶段时，由当时唯一重要的 master 分支切出。  
相比 master 分支，develop 的新 commit 是在一个开发周期内要用的，不能存储多个周期的 commit。
develop 是最全的公共分支。  
由于进 develop 时都是要做 rebase 的，所以 develop 可以在合适的时机做 rebase，保证提交历史清晰。

### feature 分支

feature 是开发者独享的临时分支，从 develop 切出来，在合适的时候 merge 回 develop。  
**不准备在当前周期发布的 feature 分支不要 merge 到 develop。**  
应该尽量避免同时开发**多个**互相有影响的 feature 分支，要么合并就在一个 feature 开发。  
如果是有全局影响的 feature，应该保证其他 feature 有办法在全局 feature 分支提交到 develop 上后 rebase，而不产生颠覆性改变。  
feature 分支进 develop，应该总是以 develop 最新的 commit 为起点，但不是 fast forward 的（no-ff），
即总是创建一个 merge commit，方便以后撤销这个 feature。  

如果某个 feature 是实验性的，并且有办法轻易的被开启或关闭，则应该在代码里面加上 feature 开关。  

某个 feature 被 revert 后，是不能重新 merge 的，需要 revert 上次的 revert。git revert -m 1 commitid // m 的作用是指那个分支是主分支  

如果 feature 分支进 develop 后，要 fix bug？  
原先的 feature 分支可以删除了，直接在 develop 上修复，并指明是那个 feature。

### release 分支

release 是为发布准备的临时分支，从 develop 切出来，供测试人员测试，同时修复测试期间的 bug，需要再往 develop 分支 merge。(git cherry-pick 或 git merge) （如果工作重心完全在 release 分支，则不妨等到 release 分支结束时再 merge，避免多次 merge。）
以 release 分支切出为起点，develop 分支可以接受下一个周期的 feature 分支，但在当前周期结束后再添加会更简单。

### master 分支

master 分支在 release/hotfix 分支测试完成以后，接收 release/hotfix 的 merge，作为所有发布过的 commit 的分支。并将发布过的 release/hotfix 打上 tag。  
如果发生重大 bug 不能立刻解决，应该将 master 重置到上次 release 的 tag。

### hotfix 分支

如果发布一段时间后，要修复线上 bug，则从 master 切出 hotfix 临时分支。hotfix 发布完毕后，将 hotfix 合并进 master 分支。
并将 hotfix 的 commit 根据需要 merge 到 develop 分支。

## git flow 不易理解的部分

1. develop 分支并不是供开发初始阶段使用的，它实际上是供发布的，进 develop 就应该有“已经做好了，这个周期就发布”的谨慎心态。
1. 真正专门供开发使用的是 feature 分支，feature 分支可以是试验性的，可以随时丢弃、改写、准备发布到 develop 或者不发布的。
1. develop 和 master 可能大不一样，commit 不一样，内容不一样，master 是 develop 的一个线上发布的里程碑记录分支。
1. develop 是长期的基础分支，master 是长期的线上发布分支，其他分支都应该定时清理。
1. 来回在 develop/release/hotfix 分支上 cherry-pick/merge 是非常痛苦的，提交历史也乱，同时也容易遗漏，应该在 feature 分支时就考虑充分，使得 cherry-pick/merge 的次数最少。
1. 减少 cherry-pick/merge 的其中一个办法是，在测试初期直接用 develop 分支测试，这样就少了大量在 release 到 develop 的 cherry-pick/merge。

## 什么是好的习惯？

很多时候大家是不知道做了那些事，会产生什么好处，这样才导致大家都便宜行事。

1. 在 feature 阶段就保证代码的质量。 一个 feature 分支一次 merge 进 develop 不出 bug，减少 feature revert，以及保留后续代码被修改的分支来源。
1. feature 分支如果可以 squash 成一个或少数 commit，就在进 develop 分支上合并 commit，方便后续分析和 revert/cherry-pick 操作。
1. merge 进其他分支时，一定要先做 rebase，并测试，再 merge 进其他分支。代码历史更清晰。
1. 每次提交都是相关的一个 feature（只做一件事），不在乎修改多少文件，比如重命名方法名可能会修改很多文件。问题的关键在于一个 commit 可以被整体 revert。
1. commit message 要言之有物。
1. 不在 feature 分支的 commit，应该加上前缀 feature-fix release-fix hot-fix
1. 不要改写公共分支。改写公共分支一定要有充分的理由，并且让其他人可以正常操作。
1. 某个 feature 在 develop 撤销时，有 develop 分支修改权限的人，可以通过 rebase 操作，将这个 feature 从 develop 完全清除。
1. 新做一个 feature 的时候，要考虑这个 feature 有没有可以不启用后撤销，如果不启用，要考虑不启用的开关。测试时，必须测试开关打开和关闭两种状态。必要时，将这个开关放置在管理后台UI控制。

## 分支与角色

1. 开发人员使用 feature/release/hotfix 分支，PR 进 develop。
1. 测试人员使用 release/hotfix 分支测试。使用 master 分支做灰度测试和线上验证。
1. 发布人员使用 develop，创建 release/hotfix 分支，合并进 master 分支。

## 分支与环境

**环境指向的分支应尽量稳定**

1. 开发人员本机使用 feature 分支，其他部分使用 develop 代码。
1. 开发环境使用 develop 分支。在特殊的情况下（比如只能在服务器上测试的代码），可以指向 feature 分支。(很多公司没有多套环境供跑 feature 分支)
1. 测试环境使用 release_v* / hotfix_v* 分支。每次发布周期进入测试阶段时，都新建测试分支，并把测试环境的分支指向 release_v* 。 （变化，release_v* / hotfix_v\* ）
1. 预发布环境使用 release 分支或 hotfix 分支。（变化，release_v* / hotfix_v* ）
1. 生产环境使用 master 分支。（不变）

## 讨论

### 未发布的 feature 在 develop 有 bug 或不用的话，怎么办？

不 revert，而是 rebase 掉。这样操作首次麻烦一些，但是方便后续 feature 再 merge 进来。但要注意 develop 是单人操作才行，这样不会相互影响。  
次级的方案是再次 PR 进 develop 上修复，这样的代价是多次修复的话，commit 历史会很支离破碎。适合公共修改的 feature 分支，因为可能其他分支已经依赖 merge 进 develop 的这个公共修改的分支了。  
再次一点的方案是 revert 上次 feature 的 merge，这样的话可以马上发布。以后要再进入这个 feature，可以 revert 上次的 revert。这个方案适合已经发布过的 feature。

### 清理 develop

因为每次进 feature 进 develop 都要 rebase，所以 develop 分支在 release 前可以做 rebase，保证 feature 的分支都在一块，或者 squash。

```sh
# 保留 merge commit
git rebase --rebase-merges
```

### 测试早期使用 develop 分支

如果不做并行版本开发，（即 release_v1.0 正在测试和修复中，develop 已经合并了 2.0 的 feature 了），那么测试期间可以直接使用 develop 分支。  
即把 release 分支简化掉。或者 release 分支存在的时间较短，仅当测试人员认为很成熟时，再切 release 分支，这样的好处是有问题可以直接在 develop 分支上解决，不用把 release 分支的 commit 再搬到 develop 上。

### 测试期间新加需求

在测试期间新加 feature 是不好的做法，但如果一定要加，那么 feature 分支像往常一样进 develop，然后再 rebase 进 release 分支。

### 其他 git 操作流程

#### 单主干的分支实践（Trunk-based development，TBD）

"TBD 的特点是所有团队成员都在单个主干分支上进行开发。当需要发布时，先考虑使用标签（tag），即 tag 某个 commit 来作为发布的版本。如果仅靠 tag 不能满足要求，则从主干分支创建发布分支。bug 修复在主干分支中进行，再 cherry-pick 到发布分支。"  
如何保证进 trunk 的就可靠，特别是公共库更新后？

#### GitHub flow

GitHub flow 是从 master 切出分支开发，然后再 merge 回 master，这种非常简单且发布频率高，但非常依赖开发质量和自动化测试。  
本文 git-flow 适合集中开发、集中测试的流程，对开发人员的依赖稍低，对测试人员的依赖高，仅当测试人员通过后，代码才能发布。

## 常用命令列表

```sh
# 新建分支(从develop分支切出)
git branch feature-login develop

# 删除分支
git branch -d feature-login

# 推送分支
git branch -u origin feature-login


# 以最新的develop来rebase feature分支
git fetch origin develop
# 注意 rebase 与 merge 不一样。merge 是把两个分支的差异当做两个commit，只解决一次冲突，就产生一个merge commit。
# rebase 在产生的冲突时，接收了 current changes 的话，那么 incomming branch 再有commit在相同行更改时，会需要再次解决冲突。
# 注意 develop 为 current branch (是以它为起点，replay feature 分支上的commit); feature 为 incomming branch
# 所以 feature 分支可以先 squash 成少量的commit，后续就减少冲突了
git rebase origin/develop feature-login

# 将feature合并到develop分支
git checkout develop
# 用远程的develop覆盖本地的develop，在远程develop被改写的情况下有用
git reset --hard origin/develop
git merge --no-ff feature-login
git push origin develop

# 删除远程分支
git push origin --delete feature-login

# 开始 release
git checkout -b release-v1.0.0 develop
# develop 中也会存在这个tag，分割不同周期的release。只需在开始release时执行一次
git tag start-release-v1.0.0
git push origin release

# 开始生产环境发布
git checkout master
git merge --no-ff release-v1.0.0
git push origin master
git tag release-v1.0.0
git checkout develop
# 将release的提交合并回develop分支，可能会有冲突，根据具体情况挑选develop或release的代码。是相同文件相同内容的块(一行或多行连续变动)不会有问题。相同文件有的块更改内容一致，有的块更改内容不一致，内容不一致的块会提示有冲突。
git checkout develop
# git merge时并不是replay commit，而是找到相同的parent commit，在这个commit的基础上看两个分支的内容块差异，最后对内容块的差异标记冲突，如果没有冲突，则直接merge成功。
git merge --no-ff release-v1.0.0
git push origin develop

# 开始 hotfix
git checkout -b hotfix-v1.0.1 master

# 完成 hotfix
git checkout master
git merge --no-ff hotfix-v1.0.1
git push origin master
git checkout develop
git merge --no-ff hotfix-v1.0.1
git push
```

## References

[A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
