---
title: 构建 Stellar 应用程序
---
在 Stellar 上构建有趣的应用吧！此列表列出了一些想法，请随意添加您的想法，或是从中挑选一个开始完成它。如果在构建应用的时候碰到了麻烦，您可以通过 [Slack chat](http://slack.stellar.org/)、[IRC](https://kiwiirc.com/client/irc.freenode.net/#stellar-dev) 或电子邮箱 developers@stellar.org 联系我们。

如果您并不想开始一个新的项目，只是想帮助我们完善已存在的应用。您可以在我们的代码仓库中寻找具有 `help wanted` 标签的 Issues。

# Slack 机器人
- 在一个频道中展示 Stellar 上源源不断的事务
- *高级功能*：允许用户向 Slack 中的其它用户发送资产/积分。例如 `/send @bob $5`。

# API 混搭
- Twilio 与 Stellar：短信交易提醒（您可以查看[这个示例](https://github.com/stellar/stellar-sms-client)）
- Twitter 与 Stellar：使用 Tweet 发送资产或使用 Twitter 告警
- Reddit 与 Stellar：小费机器人
- 更多其它可能的组合

# Horizon 数据图
一个相对简单的项目，图形化地显示从 Horizon 获取的信息，可以查找账户和事务。如果能看到以下数据就更酷了：
 - 因为任何一个账户都是被其它账户创建的，所以您可以使用树状图来展示账户间的关联。
 - 总账头部信息随时间变化的图表：
   - 事务总数
   - 操作总数
   - 总账关闭时间
   - 费用池

# 联邦服务
实现一个简单的[联邦服务](https://www.stellar.org/developers/guides/concepts/federation.html)并建立一个网站，任何人都可以在这申请一个 `名称*[yourdomain.com` 这样的 Stellar 地址，并将这个 Stellar 地址与他们的账户 ID 绑定。您也可以只为那些将[通货膨胀目标](https://www.stellar.org/developers/guides/concepts/inflation.html)设置为您的账户提供服务。

您还可以向 Stellar 基金会维护的[联邦服务](https://github.com/stellar/go/tree/master/services/federation)贡献代码。

# 发送 Lumens 到任意邮箱地址
允许任何人在 Stellar 客户端中将 Lumens 发送到任意电子邮件地址。他们只需要输入类似 `<emailaddress>*domain.com` 这样的地址就可以将 Lumens 发送过去。如果接收方还没有一个与该邮箱地址绑定的 Stellar 账户，该用户就会收到一封邮件告知他收到了 Lumens。

如果您想在 `domain.com` 上运行一个这样的服务，请执行以下步骤：

- 运行一个联邦服务。
- 将 `jed@stellar.org*domain.com` 这样的地址与 Stellar 地址关联。
- 如果您收到了一个有效的联邦服务查询申请，但是您没有找到与之对于的 Stellar 地址：
  - 生成一对 `Keypair`
  - 将生成的公钥作为 账户 ID 返回
  - 观察网络以确定该账户是否被创建了
  - 如果创建了该账户，则向给定的电子邮件地址发送一封电子邮件，邮件中其中包含密钥，点击密钥将会跳转到 Stellar 客户端的页面。

*高级功能* 允许人们通过向 control@domain.com 发送电子邮件来管理您为他们创建的账户。 使用户的邮箱成为了一个 Stellar 客户端。例如 `send 100 XLM to bob@gmail.com`。

[将这个功能添加到钱包当中](https://galactictalk.org/d/37-project-idea-sending-lumens-to-any-address)

# 分布式交易所
请在[此处](https://galactictalk.org/d/26-project-idea-distributed-exchange)查阅关于这个话题的描述与讨论。

# 资源付费墙
假设您有一个面向公众的服务，比如流媒体或者开放的 WIFI。如果用户想使用您的服务，那么需要支付小额的费用。这些付款可防止滥用或作为您的运营经费。您需要要一个服务来帮您收取费用。

## 收费服务
一个简单的服务，用于跟踪任何发送到`收费地址`的 XLM，并将它记录在数据库中，这个数据库保存了对方的公钥地址和发送的 XLM 的数量。

收费服务拥有一个可以调用的 RPC 接口：

  - `charge(公钥, XLM 数量)` 返回
    - `收取的 XLM 数量`
    - `XLM 余额`

您的应用可以公布其 Stellar 收费地址。当有人试图使用您的服务时，服务器让他们使用公钥进行身份验证，随后调用收费服务上的 `charge` 接口以减少 DB 中的使用者的余额。您可以在使用者的账户余额为零时向它发送通知。

# 多重签名协调服务
该服务帮助您更轻松的创建需要多重签名的事务。通常情况下，您必须在多方之间进行协调才能生成一个需要多重签名的事务。该 Web 站点使这个过程更加容易，并可以在您不了解另一方的情况下进行协调。

理想情况下，多重签名协调服务包含以下功能：
- 将电子邮件地址与公钥关联起来
- 创建一个需要多方签名的事务
- 输入要签署事务的公钥
- 如果这些公钥中的任何一个在之前已经关联了电子邮件地址，那么用户会收到邮件通知
- 当您访问这个网站时，您会看到一个等待您签名的事务清单：
  - 您可以看到每个事务的详细信息
  - 您可以看到是谁发起了事务
  - 您可以看到还有谁签署了事务
  - 您可以为等待您授权的事务签名
-  一旦某个事务得到了足够多的签名，它就会被提交到网络上
- 一旦提交了事务，就会通知所有的签名者

# 交易市场数据订阅(Feed)接口
为 Stellar 内置的分布式交易所提供像 [Poloniex API](https://poloniex.com/public?command=returnTicker) 这样的数据订阅接口。这对于像 [stellarTerm](http://stellarterm.com/) 这样的应用程序非常有用，并且可以将 Stellar 交易量添加到 [CoinMarketCap](http://coinmarketcap.com/) 这样的图表网站。

# 仲裁节点(Quorum)监视器
显示网络中仲裁节点状态的网页。它应该包含以下功能：
- 网络连接状态的实时图
- 哪些服务器有问题
- 任何与网络其余部分不一致的服务器
- 记录每个验证节点的正常运行时间

您应该能够从任何给定的验证验证节点的角度来查看仲裁节点状态图。您或许需要运行 stellar-core 来构建此监视器。您可以从 stellar-core 日志和 `/quorum` 命令获取数据。

*高级功能*：构建连接到 stellar-core 的服务，并监视外部消息和各个验证节点的广播。

# SDK
您可以使用您喜欢的语言构建一个 SDK：
- C#
- PHP
- Haskell
- 其它语言

也可以向已存在的 SDK 贡献代码：
- [JavaScript](https://github.com/stellar/js-stellar-sdk)
- [Go](https://github.com/stellar/go)
- [Java](https://github.com/stellar/java-stellar-sdk)
- [Python](https://github.com/StellarCN/py-stellar-base/)

# 我们了解的产品和服务理念

- 为学费、健康、保修创建的小额账户
- 小额保险
- P2P 借贷
- 有条件的现金转移
- 非营利组织的捐赠系统
- 忠诚客户积分计划
- 共同体货币
- 时间银行
- 志愿者服务时间跟踪
- 无处不在的 ATM 或 ATM 手机应用

# 促进原子跨链交换
- 最终用户软件，用于促进 Stellar 中的资产（Lumens 和 Stellar 中的其它资产）与其它生态中的加密资产进行原子跨链交换。
- 建立依托原子跨链交换技术的交易所。