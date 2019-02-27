---
title: 测试网络
---

测试网络(Testnet)是由 Stellar 开发基金会(SDF)为开发人员准备的一个小型 Stellar 测试网络。

SDF 在测试网络中运行 3 个 Stellar Core 验证节点。

您可以让 [stellar-core](https://github.com/stellar/stellar-core) 使用[此配置](https://github.com/stellar/stellar-core/blob/master/docs/stellar-core_testnet.cfg)来将其连接到测试网络。

SDF 还提供一个与测试网络相连接的 Horizon 实例。

## Stellar 测试网络适合用来做什么？

* [创建测试账户](../get-started/create-account.md) (由 Friendbot 提供测试资金)。
* 在 Stellar 上开发应用程序和探索教程，而不会丢失任何有价值的[资产](assets.md)。
* 测试 [Stellar Core](https://github.com/stellar/stellar-core/releases) 和 [Horizon](https://github.com/stellar/go/releases) 的新版本或候选版本。
* 在一个与公共网络相比数据量极小的网络中进行数据分析。

## Stellar 测试网络不适合用来做什么？

* 压力测试。
  * 如果您想对性能进行测试的话，我们推荐您阅读 [Stellar Core 的性能说明](https://github.com/stellar/stellar-core/blob/master/performance-eval.md#networks-to-test-against)。
* 测试高可用性基础架构 - SDF 无法保证测试网络的可用性。
* 在网络上长期存储数据 - [测试网络上的数据不会永久保存，会被定期重置](test-net.md#periodic-reset-of-testnet-data)。
* 测试软件，需要对测试环境做更多的适应性处理，例如:
  * 控制测试网络数据被重置的情况。
  * 在将私密数据发送到公共网络上之前，您需要对它们进行妥善的处理。

如果不想使用 SDF 提供的测试网络的话，您可以自己搭建一个私有的测试网络。

## 使用测试网络的最佳实践

### 测试网络的数据会被定期重置
为了让开发人员的拥有良好体验，SDF 测试网络会定期将总账重置为初始总账。这样清理网络，删除垃圾交易，最大限度地缩短其它节点获取最新总账的时间，并且可以使维护系统更加轻松。

当总账被重置为初始总账时，Stellar Core 和 Horizon 的所有总账条目(如账户、信任线、交易挂单等)、事务和历史数据都会被清除。这也是开发者为何不可依赖测试网络账户和余额状态的原因。

重设后，您需要[几个步骤](https://github.com/stellar/packages#testnet-reset)来重新加入和同步到测试网络。该文档也包含如何使用core和horizon的ubuntu安装包的说明。

恒星基金会会尽量无痛升级，并会至少提前两周在 [Stellar Dashboard](http://dashboard.stellar.org/)以及其它恒星社区上宣布。


从 2019 年 1 月开始，测试网络将每季度(每三个月)重置一次，暂定为：

* 二月27日
* 四月
* 七月
* 十月

测试网络会在宣布日期的UTC9点时间进行重设。

### 自动化测试

由于大多数应用程序依赖于网络中已存在的数据来执行后续的操作，因此我们建议您在编写自动测试软件时，应该让它包含自动填充数据的功能，这样您才能从容的面对测试网络被重置的情况，而且如果您选择这样做的话，您可以低成本的将该测试软件应用在私有测试网络中。

例如，如果你想要：
* 生成资产发行人以测试钱包的开发。
* 生成挂单信息以开发和测试交易客户端。

作为应用程序的维护者，您需要考虑创建一个足以代表测试主要用例的数据集，以便在测试网络不可用时也能进行可靠的测试。

您的测试软件中应包含一系列的脚本来自动帮您完成数据的填充，比如通过脚本[自动使用 Friendbot 创建账户](../get-started/create-account.md)或提交一系列的[事务](transactions.md)。

如果您还有其它问题的话，我们推荐您访问 [Stellar's Stack Exchange](https://stellar.stackexchange.com/)。
