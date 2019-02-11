---
title: 自定义资产
---

想要在 Stellar 网络发行资产或令牌的话，您需要维护三个账户。第一个是源账户，源账户即想要创建新令牌的实体账户。第二个是发行账户，发行账户由源账户创建，而新令牌是由发行账户创建的。第三个是分发账户，实体使用此账户将新令牌分发给公众。

大多数的 Stellar 文档都以诸如锚点这样的金融机构为中心。锚点是一个实体，充当货币和 Stellar 网络之间的桥梁，锚点需要设置合规服务器和桥接服务器等系统。如果您只是想在 Stellar 网路上发行资产或令牌的话，那么没有必要成为一个锚点。

以下是创建自定义令牌所需事务的详细信息。事务可以直接通过 API 提交，也可以使用 [Stellar Laboratory](https://www.stellar.org/laboratory/) 执行。


## 步骤

#### 事务 1：创建发行账户
**发起事务的账户**：源账户  
**操作**：
- [Create Account](../concepts/list-of-operations.md#create-account): 在网络中创建发行账户。
	 - starting balance: [账户最低余额](../concepts/fees.md#minimum-account-balance) + [提交事务所需的手续费](../concepts/fees.md#transaction-fee)

**事务的签署者**：源账户

#### 事务 2：创建分发账户
**发起事务的账户**：源账户
**操作**：
- [Create Account](../concepts/list-of-operations.md#create-account): 在网络中创建分发账户。
	 - starting balance: 包含了创建信任线所需的[账户最低余额](../concepts/fees.md#minimum-account-balance)

**事务的签署者**：源账户


事务 1 和事务 2 由源账户创建并提交给网络。在这一步您将创建发行账户和分发账户。您需要给这两个账户发送适量的 Lumens 以使它们满足最低账户余额的要求，对于发行账户来说，您不需要为它设置任何子条目。对于分发账户来说，您需要为它设置一条信任发行账户的信任线，所以它包含一个子项目。您还应该为它们提供少量资金，使它们能够支付提交事务产生的手续费。


#### 事务 3：添加信任线
**发起事务的账户**：分发账户  
**操作**：
- [Change Trust](../concepts/list-of-operations.md#change-trust): 添加一条包含了发行账户的信任线
	 - asset: 资产
	 	- code: 资产代码
	 	- issuer account: 发行账户
	 - trust limit: 令牌的最大发行量  

**事务的签署者**：分发账户


事务 3 由分发账户提交到 Stellar 网络，这个事务会为分发账户创建一条包含了发行账户的信任线，也就是说使分发账户信任发行账户。目前自定义资产代码有两种类型：最多包含 4 位字符的类型(Alphanumeric 4)与最多包含 12 位字符的类型(Alphanumeric 12)。字符集可以为 [a-z][A-Z][0-9]。在这一步中，您已经在 Stellar 网络中引入了您想发行的自定义令牌，但是您还没有创建任何可以用来交易的令牌。`trust limit` 参数限制了分发账户最多能保留的令牌数，因此建议将此参数设置为您想发行的令牌总数，或将其设置为账户可以容纳的最大值（最大值为 9223372036854775807(int64) 个 stroops）。


#### 事务 4：创建资产
**发起事务的账户**：发行账户  
**操作**：
- [Payment](../concepts/list-of-operations.md#payment): 向分发账户发放令牌
	 - destination: 分发账户
	 - asset: 资产
	 	- code: 资产代码
	 	- issuer account: 发行账户
	 - trust limit: 将被创建的令牌总量

**事务的签署者**：发行账户

事务 4 由发行账户创建并提交到 Stellar 网络。在这个事务中，发行账户在网络中创建了令牌，并将这些令牌分发给了分发账户。分发账户收到的令牌数量就是发行账户发行的令牌数量。

#### 事务 5：创建资产
**发起事务的账户**：发行账户  
**操作**：
- [Set Option - Home Domain](../concepts/list-of-operations.md#set-options): 设置主域名以查找 stellar.toml
	 - home domain: 域名地址

**事务的签署者**：发行账户


事务 5 由发行账户创建并提交到 Stellar 网络。`home domain` 值中的域名应该是您的 stellar.toml 文件所在的域名，stellar.toml 中包含了您资产的元信息。

在此步骤中，必须创建 stellar.toml 并将其您选定的域名上。stellar.toml 文件应包含令牌的元数据。维护 stellar.toml 文件非常重要，因为它为资产提供了详细的描述。对于每个发行的资产，stellar.toml 中的标准声明应包含以下字段（大括号中的值您需要根据实际情况填写）：
```
[[CURRENCIES]]
code="{资产代码}"
issuer="{发行账户的公钥}"
display_decimals={整数}
```

display_decimals 字段表示客户端（钱包，交易所等）应在其用户界面上显示的最大小数位数。

您也可以在 stellar.toml 中设置另外一些字段：
```
name="{名称}"
desc="{资产描述}"
conditions="{资产的使用和分配条件情况}"
image="{资产图像的 URL 地址}"
```

#### (可选) 事务 A: 限制发行资产的供应量
**发起事务的账户**：发行账户  
**操作**：
- [Set Option - Thresholds & Weights](../concepts/list-of-operations.md#set-options): 移除主密钥
	 - master weight: 0
	 - low threshold: 0
	 - medium threshold: 0
	 - high threshold: 0 

**事务的签署者**：发行账户

事务 A 由发行账户创建并提交给网络。通过将权重和阈值全部设置为零，可以创建锁定方案。所有密钥(包括帐户的主密钥)都将成为无效密钥。锁定账户意味着此账户无法再提交任何事务，因此也就不能再创建令牌。此事务的[XDR 形式的数据](https://www.stellar.org/developers/horizon/reference/xdr.html)可以在提交后显示，您可以将它展示给用户以证明发行账户已经被锁定了。

***警告：执行此步骤后，您无法再使用该发行账户创建和提交新事务。事务 A 将是这个账户能执行的最后一个事务。***


#### 事务 6：资产分发
**发起事务的账户**：分发账户  
**操作**：
- [Manage Offer - Sell](../concepts/list-of-operations.md#manage-offer): 创建一个销售您资产的订单
	- selling: 之前创建的资产
		- code: 资产代码
		- issuer account: 发行账户
	- buying: 资产
		- code: 资产代码
		- issuer account: 发行账户
	- amount: 待出售的资产的数量
	- price: 售价
	- offer id: 0  

**事务的签署者**：分发账户

事务 6 由分发帐户创建并提交给网络。在这个步骤中，创建的资产将被出售以兑换为另外一种不同的资产。不同的资产可以是另一种被创造出来的资产、法定货币、加密货币或 Lumens。如果 `offer id` 设置为 `0`，则创建一个新订单。`price` 指的是每售出一个待售资产可以得到多少个待购资产，`amount` 指出的是代售资产的数量。

通过提交事务 6，创建的令牌将会在 Stellar 的分布式交易所中交易。想要在 Stellar Term 和 Stellar Port 等交易所客户上市，请参阅他们的网站以获取详细的说明。我们鼓励您在各大交易所上市您的资产，这样可以提高您所持有资产的知名度。

## 其他示例：
想要了解与发行资产相关的更多信息，请参见[此处](../issuing-assets.md)。此外，通过[这篇文章](../concepts/assets.md#anchors-issuing-assets)，您可以更加深入地了解一些与创建资产有关的术语。[这篇文章](https://www.stellar.org/blog/tokens-on-stellar/)会教您如何简单使用 Stellar Laboratory 创建令牌。

## 资源:
- [成为一个锚点](../anchor/) - Stellar.org
- [计算账户需要保留的最低余额](../concepts/fees.md#minimum-account-balance) - Stellar.org
- [概念：stellar.toml](../concepts/stellar-toml.md) - Stellar.org
- [概念：信任线](../concepts/assets.md#trustlines) - Stellar.org
