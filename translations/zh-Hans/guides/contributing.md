---
title: 贡献指南
---

# 如何为 Stellar 项目做出贡献

您对恒星网络的贡献将帮助我们更快地改善世界金融基础设施。

我们希望用户能够轻松地提交一个修改，以帮助 Stellar 网络茁壮成长。以下是我们要求贡献者遵循的一些指导原则，以便我们能够快速地合并您的更改。

## 如何开始

* 您需要有一个 [GitHub 帐号](https://github.com/join)。
* 如果 GitHub 上没有与您问题相关的 Issue，那么您需要自己创建一个。
  * 清晰且详细地描述这个问题，如果是一个 bug 的话请写出如何复现它。
* Fork Github 上的代码仓库。

## 找寻您可以参与的问题

您可以现在看看您感兴趣的项目的 GitHub Issue 列表，那些被标记为 [help wanted](https://github.com/issues?q=is%3Aopen+is%3Aissue+user%3Astellar+label%3A%22help+wanted%22) 的问题通常来说是一个好的切入点。

我们也使用 GitHub Issues 来标记和跟踪正在处理的问题，如果您看到某个 Issue 被标记为 `in progress`，这意味着有其他人正在处理这个问题。`orbit` 标签意味着我们可能会在未来一两周处理这个问题。`ready` 标签意味着我们已经将它标记为高优先级，将尽快开始处理它。

如果您认为有些东西需要添加或修改，请添加或参与一个 Issue。

## 更改

* 以您想作出更改的分支为基础创建一个独立的主题分支，您将在这里进行修改。
  * 您想修改的分支一般是 master 分支。
  * 请避免直接在 `master` 分支进行修改。
* 请确保您为您修改的代码添加了相应的测试，并且测试通过了。

## 提交更改

* [签署贡献者许可协议](https://docs.google.com/forms/d/1g7EF6PERciwn7zfmfke5Sir2n10yddGGSXyZsq98tVY/viewform?usp=send_form)
* 所有的内容、评论和 pull requests 必须遵循 [Stellar 社区指南](https://www.stellar.org/community-guidelines/)。
* 将更改推送到您 fork 的仓库的一个主题分支中。
* 向 Stellar 提交一个 pull request。
 * [提交信息(commit message)](https://github.com/erlang/otp/wiki/Writing-good-commit-messages)应该描述您做了什么事情。
 * 一个通过 pull request 提供的更改应该只关注一个问题(issue)。
 * 确保您的 pull request 与当前代码库没有冲突。

之后您需要等待我们的审核。我们会在三个工作日之内（通常是一个工作日）对 pull requests 进行评论。我们可能会提出一些改变、改进或替代方案。

## 微小的更改

### 文档
对于注释和文档的小改动，并不总是需要创建新的 GitHub Issue。在这种情况下，请以 "doc" 作为提交消息的开头。

# 其他资源
* [贡献者许可协议](https://docs.google.com/forms/d/1g7EF6PERciwn7zfmfke5Sir2n10yddGGSXyZsq98tVY/viewform?usp=send_form)
* [探索 API](https://www.stellar.org/developers/reference/)
* [Slack](http://slack.stellar.org) 上的 #dev 频道
* freenode.org 上的 #stellar-dev IRC 频道


本文的灵感来自：

https://github.com/puppetlabs/puppet/blob/master/CONTRIBUTING.md

https://github.com/thoughtbot/factory_girl_rails/blob/master/CONTRIBUTING.md

https://github.com/rust-lang/rust/blob/master/CONTRIBUTING.md