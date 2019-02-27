---
title: Stellar 网络概览
---

![Stellar 生态](https://www.stellar.org/wp-content/uploads/2016/06/Stellar-Ecosystem-v031.png)

使用 Stellar 网络，您可以构建数字钱包、银行工具、自动扣费的智能设备，以及您可以想到的任何其他涉及付款的东西！尽管 Stellar 是一个复杂的分布式系统，但使用它并不复杂。

## API: Horizon

**大多数应用程序通过 [Horizon](https://www.stellar.org/developers/horizon/reference/) 与 Stellar 网络交互，** Horizon 是一个 RESTful HTTP API 服务器。Horizon 为您提供了一个直接的方式来提交事务、检查账户或订阅事件。 由于它使用的是 HTTP 协议，因此您可以使用网络浏览器、简单的命令行工具（如 cURL），或者您喜欢的编程语言中的 Stellar SDK 与 Horizon 进行交互。

安装 Horizon 最简单的方法是使用 [**stellar/quickstart** Docker 镜像](https://hub.docker.com/r/stellar/quickstart/)。

Stellar.org 官方维护了用于与 Horizon 交互的 [JavaScript](https://github.com/stellar/js-stellar-sdk)、 [Java](https://github.com/stellar/java-stellar-sdk) 和基于 [Go](https://github.com/stellar/go/tree/master/clients/horizon) 的 SDK， 另有 [Ruby](https://github.com/stellar/ruby-stellar-sdk), [Python](https://github.com/StellarCN/py-stellar-base), 和 [C#](https://github.com/elucidsoft/dotnet-stellar-sdk) 等的社区维护的 SDK。

## 网络核心：Stellar Core

**[Stellar Core](../../stellar-core/learn/admin.html) 是整个 Stellar 网络的核心，每个 Horizon 都与它相连。** Stellar Core 软件通过 [Stellar Consensus Protocol](../concepts/scp.md) (SCP) 对每个事务的状态进行验证，并与其它的 Stellar Core 实例达成一致。 恒星网络本身是由世界各地不同的个人和实体运行相连的 Stellar Core 的集合。有些实例具有可以通信的 Horizon 服务器功能，也有些实例的存在只是为了增加整个网络的可靠性。

安装 Stellar Core 最简单的方法是使用 [**stellar/quickstart** Docker 镜像](https://hub.docker.com/r/stellar/quickstart/)。

您也许想要托管您自己的 Stellar Core 实例，通过运行自己的实例，您可以在不依赖第三方的情况下提交事务，配置您想信任的节点，还能使 Stellar 网络具有更加可靠性与稳固性。

## 综览：Stellar 网络

恒星网络是全球范围内的 Stellar Core 节点集合，每个节点由不同的人和组织维护。 网络的分布式特性使其具有可靠性与安全性。

所有的这些 Stellar Core 节点最终都对事务集达成一致。网络上的每笔交易只需支付少量费用：100 stroops (0.00001 XLM)，这笔费用有助于使网络免受恶意行为的侵害。

为了帮助您测试工具和应用，Stellar.org 部署了一个小型测试网络和 Horizon 实例。[去了解一下测试网络吧。](../concepts/test-net.md)

<div class="sequence-navigation">
  <a class="button button--next" href="create-account.html">下一页: 创建账户</a>
</div>
