---
title: 配置虹桥服务器
---

通过虹桥服务，用户可以将 BTC/ETH 移动到 Stellar 网络中。它既可以用于在 Stellar 网络上表示 BTC 或 ETH，也可以用于将其兑换为另一种自定义令牌，这个功能对 ICO(Initial Coin Offering) 特别有用。本指南将重点介绍如何设置虹桥服务器以将 ETH 转移至 Stellar 网络。

## 您需要配置哪些东西？

- PostgreSQL 数据库
- Bitcoin/Ethereum 节点
- 虹桥服务器

## 设置 PostgreSQL

由于不同的操作系统配置 PostgreSQL 的步骤不尽相同，所以我们不会在这里介绍如何配置它，请访问相关的在线文档。

## 设置 Ethereum 节点

- 下载 [geth(版本 1.7.1 或以上)](https://geth.ethereum.org/downloads/).
- 提取下载文件的内容
- 在测试网络上启动节点

```bash  
./geth --testnet --rpc
```

- 想要了解更多请阅读[管理 geth](https://github.com/ethereum/go-ethereum)

## 为您的资产创建一个出售订单

虹桥将收到的 BTC 或 ETH 自动地兑换为您的自定义令牌。为了实现这一点，必须在 Stellar 的分布式交易所中创建 CUSTOM-TOKEN/BTC 或 CUSTOM-TOKEN/ETH 资产对的卖出订单。

举例来说，我们假设 1 `TOKE` 可以兑换到 0.2 `ETH`。您可以使用 [Stellar Laboratory](https://www.stellar.org/laboratory/) 来创建并提交一个订单。

- 进入 "Transaction Builder(创建事务)" 页面
- 在右上角有一个 "test/public" 按钮，如果您想使用公共网络，请将它设置为 public，如果您想使用测试网络，请将它设置为 testnet。
- 在页面中的表格中填写以下数据：
  - 输入源账户(资产发行账户或资产分发账户)
  - 点击 "Fetch next sequence number(获取下一个序列号)" 按钮
  - 向下滚动，添加需要操作类型： "Manage Offer"
  - 待售资产的类型：Alphanumeric 4
  - 待售资产的代码：`TOKE`
  - 待售资产的发行账户：发行账户
  - 待购资产的类型：Alphanumeric 4
  - 待购资产的代码：`ETH`
  - 待购资产的发行账户：发行账户
  - 数量：您想出售的 `TOKE` 的数量
  - 价格：以购买资产表示。，即 `1 待售资产 = X 待购资产`。在我们的示例中，由于我们想要以 0.2 ETH 的价格出售 1 TOKE，因此这里的值应为 0.2
  - 订单 ID：输入 "0" 以创建一个新订单
  - 向下滚动，点击 "Sign transaction in Signer(签名者签名事务)"
  - 输入资产发行账户或分发账户的密钥来签署事务或使用 Ledger 签署事务
  - 点击 "提交事务"
  - 点击 "提交"

上面的步骤将在分布式交易所中为您的资产创建一个出售订单。

## 设置虹桥服务器

- 下载[最新的版本](https://github.com/stellar/go/releases/tag/bifrost-v0.0.2)并将它提取到一个文件夹中。
- 将下载文件重命名为 `bifrost-server` (可选)
- 使用 [BIP-0032](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 协议生成您的主公钥。您可以从 GitHub 上下载[该实现](https://iancoleman.io/bip39/)，然后在一台离线的计算机上生成密钥。您还可以在 Ledger 上生成您的主公钥。
- 创建一个和以下配置文件的类似的配置文件，文件名为 `bifrost.cfg`：

<code-example name="bifrost.cfg">

```toml
port = 8002
using_proxy = false
access_control_allow_origin_header = "*"

#如果您想接收 BTC 的话，请将这些注释符删除。
#[bitcoin]
#master_public_key = "xpub6DxSCdWu6jKqr4isjo7bsPeDD6s3J4YVQV1JSHZg12Eagdqnf7XX4fxqyW2sLhUoFWutL7tAELU2LiGZrEXtjVbvYptvTX5Eoa4Mamdjm9u"
#rpc_server = "localhost:18332"
#rpc_user = "user"
#rpc_pass = "password"
#testnet = true
#minimum_value_btc = "0.0001"
#token_price = "1"

[ethereum]
master_public_key = "xpub68VNckQn96Y23e5GsGh9X7zVmbPT4ho5Vdf6RdgMGG3LyNhH2cLFDCib9zgn8QWgj261xu7MYbmBsX8Fp5VkfDUrecUnpEGWkyCo7qK2gxn"
rpc_server = "localhost:8545"
network_id = "3"
minimum_value_eth = "0.00001"
token_price = "1"

[stellar]
issuer_public_key = "GDNPOP72ZO6AZXZ7LQJ4GKYT7UIH4JEG4X3ZRZBFUCRB467RNV3SFK5D"
distribution_public_key = "GCSSFPPVERDH4ZPWH5BSONEJERHCVS4DPZRWJG3FP3STOA5ZFTD3GMZ5"
signer_secret_key = "SB3WH2NLOFW2K2B5MWN34CWF35ZLQXH33ABZYL7KZFKTVEFP72Q574LM"
token_asset_code = "ZEN"
needs_authorize = false
horizon = "https://horizon-testnet.stellar.org"
network_passphrase = "Test SDF Network ; September 2015"
starting_balance = "4"

[database]
type="postgres"
dsn="postgres://stellar:pass1234@localhost/bifrost?sslmode=disable"
```

</code-example>


- 使用[此处](https://github.com/stellar/go/tree/master/services/bifrost#config)描述的值完成配置文件
- 运行以下命令检查您是否拥有正确的主公钥：

```bash 
./bifrost-server check-keys
```

输出应类似于：

```bash
MAKE SURE YOU HAVE PRIVATE KEYS TO CORRESPONDING ADDRESSES:
Bitcoin MainNet:
No master key set...
Ethereum:
0 0xAF484B67cC184259d22edfA4aFe874f68275B714
1 0x0163DF805B87A9aB2dd3177f674B275163272630
2 0x42069115ba5802736444Aacba5F0bD4a9a007E69
3 0xA219bCCFeE13B94fcf505120Cb7b8CD090749A4e
4 0x3AB571B247b0CF45E44d111691F9D03eE1bfE705
5 0x1Fe3101B058Aa3b6Fb69B84Cd1cc7766959dcFc2
6 0x1B07c658614F6D4F13225b63d76055EaB07114c9
7 0x3C3459c47388163E56e544F9616ac0E46668420E
8 0x08fb48e4f54f699cDa3B97cd97D9fB6A594354D7
9 0xC5CD4b9E6c5D9c0cd1AAe5A52f6DCA3d20CF08BC
```

## 启动虹桥服务器

当您完成了对虹桥服务的设置之后，您可以通过以下命令来启动它：

```bash
./bifrost-server server
```
虹桥服务器将负责生成以太地址，监听这些地址的收款信息并将发行的令牌发送给购买者。

## 使用虹桥 JS SDK

通过虹桥 JS SDK ，客户端可以与虹桥服务器进行通信。您可以下载[最新版](https://github.com/stellar/bifrost-js-sdk/releases)的 SDK，并将其应用在前端应用程序中。想要了解如何将它应用在前端应用中，请参阅 [bifrost-js-sdk repo](https://github.com/stellar/bifrost-js-sdk) 库中的[示例 html 文件](https://github.com/stellar/bifrost-js-sdk/blob/master/example.html)。