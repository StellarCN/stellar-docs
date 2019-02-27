---
title: 操作清单
---

如果您想要知道操作是如何在 Stellar 中运行的，请阅读[这篇文章](./operations.md)。

如果您想要了解操作的协议规范，请参阅 [stellar-transactions.x](https://github.com/stellar/stellar-core/blob/master/src/xdr/Stellar-transaction.x)。

- [Create Account(创建账户)](#create-account)
- [Payment(付款)](#payment)
- [Path Payment(路径付款)](#path-payment)
- [Manage Offer(管理订单)](#manage-offer)
- [Create Passive Offer(创建被动订单)](#create-passive-offer)
- [Set Options(设置账户选项)](#set-options)
- [Change Trust(修改信任)](#change-trust)
- [Allow Trust(允许信任)](#allow-trust)
- [Account Merge(账户合并)](#account-merge)
- [Inflation(通货膨胀)](#inflation)
- [Manage Data(管理数据条目)](#manage-data)
- [Bump Sequence(增大序列号)](#bump-sequence)

## Create Account(创建账户)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.createAccount) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/CreateAccountOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#CreateAccountBuilder)

通过这个操作给特定的账户发送一定数量的 Lumens 以在网络中创建该账户。

阈值等级：中

结果：`CreateAccountResult`

参数：

| 参数        | 类型       | 描述                                                                                |
| ---------------- | ---------- | ------------------------------------------------------------------------------------------ |
| Destination      | account ID | 需要被激活的账户地址。                                  |
| Starting Balance | integer    | 用于创建新账户的 XLM 的数量，这些 XLM 将从源账户中扣除。 |


可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|CREATE_ACCOUNT_MALFORMED| -1| `destination` 无效。 |
|CREATE_ACCOUNT_UNDERFUNDED| -2| 源账户没有足够的资产来创建这个账户，或是这个操作会导致账户无法满足它的售出负债。另外，发送 XLM 的时候需要确保发送后账户仍然满足账户最低余额的要求。 |
|CREATE_ACCOUNT_LOW_RESERVE| -3| 发送给 `destination` 账户的 XLM 数目不足以让它满足最低账户余额的要求。 |
|CREATE_ACCOUNT_ALREADY_EXIST| -4| `destination` 账户已经存在，无需重复创建。 |

## Payment(付款)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.payment) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/PaymentOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#PaymentBuilder)

将特定数量的特定资产发送给目标账户。

阈值等级：中

结果：`PaymentResult`

参数：

|参数| 类型| 描述|
| --- | --- | --- |
|Destination| account ID| 收款方的账户 ID。 |
|Asset| asset| 发送的资产类型。 |
|Amount| integer| 发送的数额。 |

可能发送的错误：

|错误代码| 编号| 描述|
| --- | --- | --- |
|PAYMENT_MALFORMED| -1| 输入的参数不正确。 |
|PAYMENT_UNDERFUNDED| -2| 源账户(发送方)没有足够的资产去完成这笔付款，或是这个操作会导致账户无法满足它的售出负债。另外，发送 XLM 的时候需要确保发送后账户仍然满足账户最低余额的要求。 |
|PAYMENT_SRC_NO_TRUST| -3| 源账户没有信任他正在尝试发送的资产的发行账户。 |
|PAYMENT_SRC_NOT_AUTHORIZED| -4| 源账号无权发送该资产。 |
|PAYMENT_NO_DESTINATION| -5| 接收方的账户不存在。 |
|PAYMENT_NO_TRUST| -6| 接收方没有信任待接收资产的发行账户。请通过[资产](./assets.md)的文档了解更多。 |
|PAYMENT_NOT_AUTHORIZED| -7| 接收方没有被资产发行方允许持有该资产。 |
|PAYMENT_LINE_FULL| -8| 接收这笔付款将会导致接收方无法满足购买负载或资产持有数量的限制。 |
|PAYMENT_NO_ISSUER| -9| 资产的发行方不存在。 |

## Path Payment(路径付款)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.pathPayment) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/PathPaymentOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#PayWithPath)

通过报价路径将特定数目的特定资产发送到目标账户。这可以使发送的资产(例如，450 XLM)不同于接收的资产(例如，6 BTC)。

需要注意的几点：
* 路径支付不允许中间报价来自源账户，因为这会产生最差的汇率。您需要将路径付款拆分为两个支付数目较小的路径付款，或者确保源账户的订单不在订单簿的顶部。
* 双方的账户余额在操作结束是计算。
   * 这里有一点值得一提，当 `(收款账户, 接收的资产) == (发送账户, 发送的资产)` 时，该账户相当于在操作执行期间获得无息贷款。

阈值等级：中

结果：`PathPaymentResult`

参数：

|参数| 类型| 描述|
| --- | --- | --- |
|Send asset| asset| 从发送账户中扣除的资产类型。 |
|Send max| integer| 允许从源账户中扣除的 `send asset` 的最大数额(不包括手续费)。 |
|Destination| account ID| 收款方的账户 ID。 |
|Destination asset| asset| 接收方收到的资产类型。 |
|Destination amount| integer| 接收方收到的 `destination asset` 的数目。 |
|Path| list of assets| 兑换路径中使用到的中间资产(`send asset` 和 `destination asset` 不包含在这里面)。举例来说，您找到了唯一一条将 USD 兑换为 EUR 的路径，这条路径经过了 XLM 与 BTC。这条路径是 USD -> XLM -> BTC -> EUR，请注意  `path`  中包含了 XLM 和 BTC。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|PATH_PAYMENT_MALFORMED| -1| 输入的参数不正确。 |
|PATH_PAYMENT_UNDERFUNDED| -2| 源账户(发送方)没有足够的资产去完成这笔付款，或是这笔付款无法满足账户的售出负债。另外，发送 XLM 的时候需要确保发送后账户仍然满足账户最低余额的要求。 |
|PATH_PAYMENT_SRC_NO_TRUST| -3| 源账户没有信任他正在尝试发送的资产的发行账户。 |
|PATH_PAYMENT_SRC_NOT_AUTHORIZED| -4| 源账号无权发送该资产。 |
|PATH_PAYMENT_NO_DESTINATION| -5| 接收方的账户不存在。 |
|PATH_PAYMENT_NO_TRUST| -6| 接收方没有信任待接收资产的发行账户。请通过[资产](./assets.md)的文档了解更多。 |
|PATH_PAYMENT_NOT_AUTHORIZED| -7| 接收方没有被资产发行方允许持有该资产。 |
|PATH_PAYMENT_LINE_FULL| -8| 接收这笔付款将会导致接收方无法满足购买负载或资产持有数量的限制。 |
|PATH_PAYMENT_NO_ISSUER| -9| 其中某个资产的发行方缺失了。 |
|PATH_PAYMENT_TOO_FEW_OFFERS| -10| 没有找到兑换 `send asset` 与  `destination asset` 的路径。Stellar 查找的路径最多只能拥有 6 个跃点。 |
|PATH_PAYMENT_OFFER_CROSS_SELF| -11| 这笔付款会使自己的订单成交。 |
|PATH_PAYMENT_OVER_SENDMAX| -12| 通过此路径发送特定`数量(destination amount)`的`目标资产(destination asset)`的会导致超出`限额(send max)`。 |

## Manage Offer(管理订单)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.manageOffer) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/ManageOfferOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#ManageOfferBuilder)

创建、更新或删除订单。

如果要创建新的订单，请将 Offer ID 设置为 `0`。

如果您想更新一个订单，请将 Offer ID 设置为您需要修改的订单的 Offer ID。

如果您想删除一个订单，请将 Offer ID 设置为您需要删除的订单的 Offer ID，并将数量设置为  `0`.

阈值等级：中

结果：`ManageOfferResult`

|参数| 类型| 描述|
| --- | --- | --- |
| Selling| asset| 订单创建者想出售的资产。 |
| Buying| asset| 订单创建者想购买的资产。 |
| Amount| integer| 需要售出的资产的数目。当您需要删除订单是请将该值设置为 `0`， |
| Price| {numerator, denominator} | Price 代表每售出一个待售资产可以得到多少个待购资产。例如，如果您想通过卖出 30 XLM 获得 5 BTC，那么 Price 可以设置为 {5,30}。 |
| Offer ID| unsigned integer| 订单的 ID。 需要新建订单请将该值设置为 `0`，需要修改和删除某个订单，请将该值设置为目标订单的 Offer ID。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|MANAGE_OFFER_MALFORMED| -1| 输入的参数不正确。 |
|MANAGE_OFFER_SELL_NO_TRUST| -2| 创建订单的账户尚未为意图出售的资产创建信任线。 |
|MANAGE_OFFER_BUY_NO_TRUST| -3| 创建订单的账户尚未为意图购买的资产创建信任线。 |
|MANAGE_OFFER_SELL_NOT_AUTHORIZED| -4| 创建订单的账户无权出售此资产。 |
|MANAGE_OFFER_BUY_NOT_AUTHORIZED| -5| 创建订单的账户无权购买此资产。 |
|MANAGE_OFFER_LINE_FULL| -6| 创建此订单会导致该账户无法满足购买负载或买入资产的持有数量的限制。 |
|MANAGE_OFFER_UNDERFUNDED| -7| 创建此订单会导致该账户无法满足买入的资产的持有数量的限制或售出负载。请注意，如果是销售 XLM 的话，那么此账户必须保证满足最低账户余额的要求，最低账户余额是假设此订单不会立即完全成交而计算的。 |
|MANAGE_OFFER_CROSS_SELF| -8| 此账号有着等价或价格更低的对手订单，如果创建此订单会导致自己与自己交易。 |
|MANAGE_OFFER_SELL_NO_ISSUER| -9| 出售资产的发行账户不存在。 |
|MANAGE_OFFER_BUY_NO_ISSUER| -10| 买入资产的发行账户不存在。 |
|MANAGE_OFFER_NOT_FOUND| -11| 提供的 `offerID` 不存在。 |
|MANAGE_OFFER_LOW_RESERVE| -12| 该账户的 XLM 余额或售出负债不足以使该账户能够创建一个订单。每创建一个订单都会增加账户需要持有的最低账户余额。 |

## Create Passive Offer(创建被动订单)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.createPassiveOffer) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/CreatePassiveOfferOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#ManageOfferBuilder)

被动订单是一种不会在相同报价的情况下成交的订单，只有在报价不相等时才会成交。
例如，使用 XLM 购买 BTC 的最佳价格为 6XLM/BTC，然后您创建了一个以 6XLM/BTC 的价格售出 BTC 的被动订单，您的被动订单将*不会*立刻成交。

值得注意的是，在被动订单之后创建的常规订单可以与该被动订单成交，即使两者的报价相同。

被动订单允许做市商有零利差。如果您想以 1:1 的价格相互兑换美元和欧元，您可以创建两个被动报价，这两个订单不会相互满足并成交。

一旦被动报价被创建，您可以像管理其它类型的报价一样使用 [manage offer](#manage-offer) 操作来管理它。

阈值等级：中

结果：`CreatePassiveOfferResult`

|参数| 类型| 描述|
| --- | --- | --- |
|Selling| asset| 订单创建者想出售的资产。 |
|Buying| asset| 订单创建者想购买的资产。 |
|Amount| integer| 需要售出的资产的数目。 |
|Price| {numerator, denominator}| Price 代表每售出一个待售资产可以得到多少个待购资产。例如，如果您想通过卖出 30 XLM 获得 5 BTC，那么 Price 可以设置为 {5,30}。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|MANAGE_OFFER_MALFORMED| -1| 输入的参数不正确。 |
|MANAGE_OFFER_SELL_NO_TRUST| -2| 创建订单的账户尚未为意图出售的资产创建信任线。 |
|MANAGE_OFFER_BUY_NO_TRUST| -3| 创建订单的账户尚未为意图购买的资产创建信任线。 |
|MANAGE_OFFER_SELL_NOT_AUTHORIZED| -4| 创建订单的账户无权出售此资产。 |
|MANAGE_OFFER_BUY_NOT_AUTHORIZED| -5| 创建订单的账户无权购买此资产。 |
|MANAGE_OFFER_LINE_FULL| -6| 创建此订单会导致该账户无法满足购买负载或买入资产的持有数量的限制。 |
|MANAGE_OFFER_UNDERFUNDED| -7| 创建此订单会导致该账户无法满足买入的资产的持有数量的限制或售出负载。请注意，如果是销售 XLM 的话，那么此账户必须保证满足最低账户余额的要求，最低账户余额是假设此订单不会立即完全成交而计算的。 |
|MANAGE_OFFER_CROSS_SELF| -8| 此账号有着等价或价格更低的对手订单，如果创建此订单会导致自己与自己交易。 |
|MANAGE_OFFER_SELL_NO_ISSUER| -9| 出售资产的发行账户不存在。 |
|MANAGE_OFFER_BUY_NO_ISSUER| -10| 买入资产的发行账户不存在。 |
|MANAGE_OFFER_NOT_FOUND| -11| 提供的 `offerID` 不存在。 |
|MANAGE_OFFER_LOW_RESERVE| -12| 该账户的 XLM 余额或售出负债不足以使该账户能够创建一个订单。每创建一个订单都会增加账户需要持有的最低账户余额。 |

## Set Options(设置账户选项)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.setOptions) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/SetOptionsOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#SetOptionsBuilder)

此操作设置账户的各种选项。

有关设置签名的更多信息，请参阅[多重签名](https://stellar-docs.overcat.me/guides/concepts/multi-sig.html)。

当设置签名者或阈值时，此操作的阈值等级为高。

阈值等级：中或高

结果：`SetOptionsResult`

参数：

|参数| 类型| 描述|
| --- | --- | --- |
|inflation Destination| account ID| 通货膨胀的目标账户。 |
|Clear flags| integer| 指定您想要清除的标识位。想要详细了解标识位，请参阅[账户文档](./accounts.md)。位掩码整数从账户的现有标识中减去。这允许在不知道现有标志的情况下设置特定标识位。 |
|Set flags| integer| 指定您想要设置的标识位。想要详细了解标识位，请参阅[账户文档](./accounts.md)。位掩码整数添加到账户的现有标识上。这允许在不知道现有标志的情况下设置特定标识位。 |
|Master weight| integer| 主密钥的权重。此账户还可以使用 `signer` 参数添加用于签署事务的其他密钥。 |
|Low threshold| integer| 0-255之间的数字，表示此账户在其执行的具有[低等阈值的](https://stellar-docs.overcat.me/guides/concepts/multi-sig.html)操作所需要的[阈值](https://stellar-docs.overcat.me/guides/concepts/multi-sig.md)。 |
|Medium threshold| integer| 0-255之间的数字，表示此账户在其执行的具有[中等阈值的](https://stellar-docs.overcat.me/guides/concepts/multi-sig.html)操作所需要的[阈值](https://stellar-docs.overcat.me/guides/concepts/multi-sig.md)。 |
|High threshold| integer| 0-255之间的数字，表示此账户在其执行的具有[高等阈值的](https://stellar-docs.overcat.me/guides/concepts/multi-sig.html)操作所需要的[阈值](https://stellar-docs.overcat.me/guides/concepts/multi-sig.md)。 |
|Home domain| string| 为账户设置主域名。请参阅[联邦服务](./federation.md)。 |
|Signer| {Public Key, weight}| 为账户添加，更新或删除签名者。如果将权重设为 0 的话，则删除签名者。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|SET_OPTIONS_LOW_RESERVE| -1| 该账户的 XLM 余额或售出负债不足以使该账户能够新增一个子条目。每添加一个新的签名者都会增加账户需要持有的最低账户余额。 |
|SET_OPTIONS_TOO_MANY_SIGNERS| -2| 账户最多可拥有 20 个签名者，当添加的签名者超过这个限制的时候会失败。 |
|SET_OPTIONS_BAD_FLAGS| -3| 添加/清除的单个标志或组合无效。 |
|SET_OPTIONS_INVALID_INFLATION| -4| `inflation` 字段中设置的目标账户不存在。 |
|SET_OPTIONS_CANT_CHANGE| -5| 该账户正在尝试更改无法再被更改的设置项。 |
|SET_OPTIONS_UNKNOWN_FLAG| -6| 该账户正在尝试设置未知的标志。 |
|SET_OPTIONS_THRESHOLD_OUT_OF_RANGE| -7| 阈值或为签名者设置的权重不合理。 |
|SET_OPTIONS_BAD_SIGNER| -8| 主密钥不能作为额外的签名者添加到账户中。 |
|SET_OPTIONS_INVALID_HOME_DOMAIN| -9| 主域名参数不正确。 |

## Change Trust(修改信任)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.changeTrust) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/ChangeTrustOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#ChangeTrustBuilder)

创建，更新或删除信任线。有关信任线的更多信息，请参阅[资产文档](https://stellar-docs.overcat.me/guides/concepts/assets.html)。

阈值等级：中

结果：`ChangeTrustResult`

|参数| 类型| 描述|
| --- | --- | --- |
|Line| asset| 某项资产的信任线。举例来说，一个用户新建了一条信任线，允许最多持有 200 个锚点发行的 USD，则 `line` 应该设置为：USD:anchor。 |
|Limit| integer| 持有的该资产的数量的限制。以上个例子为例，`limit` 应该设置为 200。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|CHANGE_TRUST_MALFORMED| -1| 输入的参数不正确。 |
|CHANGE_TRUST_NO_ISSUER| -2| 无法找到资产的发行人。 |
|CHANGE_TRUST_INVALID_LIMIT| -3| 将 `limit` 设置为该值会导致该账户无法满足购买负载或现有资产的持有数量的限制。 |
|CHANGE_TRUST_LOW_RESERVE| -4| 该账户的 XLM 余额或售出负债不足以使该账户能够新增一个子条目。每添加一个新的信任线都会增加账户需要持有的最低账户余额。 |

## Allow Trust(允许信任)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.allowTrust) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/AllowTrustOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#AllowTrustBuilder)

设置现有信任线的授权(`authorized`)标志。这个操作只能由信任线中的[资产](./assets.md)的发行账户调用，并且只有在发行账户上启用了 `AUTHORIZATION REQUIRED` 时(最少需要满足这个要求)才能调用。

发行账户只有在启用了 `AUTH_REVOCABLE_FLAG` 标识后才能清除其它用户的  `authorized` 标识，否者只能启用其它用户的 `authorized` 标识

如果发行账户清除了 `Trustor` 的 `authorized` ，则由 `trustor` 发起的所有包含该资产的订单都会被删除。*(协议版本 10 及以上)*

阈值等级：低

结果：`AllowTrustResult`

|参数| 类型| 描述|
| --- | --- | --- |
|Trustor| account ID| 设置了该信任线的目标账户。 |
|Type| asset | 信任线中的资产类型。例如，如果锚点希望允许另一个账户持有其美元资产，则 `Type` 应该设置为 USD:anchor。 |
|Authorize| boolean| 该信任线是否得到了授权。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|ALLOW_TRUST_MALFORMED| -1| 指定资产类型的 `Type` 参数非法。 |
|ALLOW_TRUST_NO_TRUST_LINE| -2| `trustor` 并没有设置包含此发行账户的信任线。 |
|ALLOW_TRUST_TRUST_NOT_REQUIRED| -3| 源账户(执行此操作的发行账户)不需要执行信任操作。也就是说源账户并没有启用 `AUTH_REQUIRED_FLAG 标识。 |
|ALLOW_TRUST_CANT_REVOKE| -4| 源账户正试图撤消对 `trustor` 的信任线的授权，但它不能这样做。 |

## Account Merge(账户合并)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.accountMerge) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/AccountMergeOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#AccountMergeBuilder)

将源账户中的所有原生资产(即源账户持有的 XLM)转移到另一个账户，并且将源账户从总账中移除。

阈值等级：高

结果：`AccountMergeResult`

|参数| 类型| 描述|
| --- | --- | --- |
|Destination| account ID| 该账户将会收到源账户的所有 XLM。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|ACCOUNT_MERGE_MALFORMED| -1| 该操作格式不正确，因为源账户无法与自身合并。 `destination` 不能是源账户。 |
|ACCOUNT_MERGE_NO_ACCOUNT| -2| `destination` 账户不存在。 |
|ACCOUNT_MERGE_IMMUTABLE_SET| -3| 源账户启用了 `AUTH_IMMUTABLE` 标识。 |
|ACCOUNT_MERGE_HAS_SUB_ENTRIES | -4| 源账户有信任线或订单。 |
|ACCOUNT_MERGE_SEQNUM_TOO_FAR | -5| 源账户的序列号过大。它必须小于 `(ledgerSeq << 32) = (ledgerSeq * 0x100000000)`。*(协议版本 10 及以上)* |
|ACCOUNT_MERGE_DEST_FULL| -6| `destination` 收到源账户的资产后将无法满足自身的 Lumens 购买负债。(协议版本 10 及以上)* |

## Inflation(通货膨胀)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.inflation) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/InflationOperation.html) | [Go](https://godoc.org/github.com/stellar/go/build#InflationBuilder)

通过该操作执行通胀。

阈值等级：低

结果：`InflationResult`

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|INFLATION_NOT_TIME| -1| 每周只能执行一次通胀，这个错误意味着还未到执行新一轮通胀的时间。 |

## Manage Data(管理数据条目)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.manageData) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/ManageDataOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#ManageDataBuilder)

允许您设置、修改或删除特定账户上的数据条目(键/值对)。一个账户可以设置任意数量的数据条目。每添加一个数据条目都会增加账户需要持有的最低账户余额。

数据对可以用于各种应用程序中，但它们并不被 Stellar 的核心协议所使用。

阈值等级：中

结果：`ManageDataResult`

|参数| 类型| 描述|
| --- | --- | --- |
|Name| string | 长度最多为 64 bytes 的字符串。如果这是一个新的键(Name)，它将向账户添加给定的数据条目。如果此键(Name)已经存在，则将修改其对应的值。 |
|Value| binary data | (可选) 如果该值为空的话，这个数据条目会被删除。如果存在的话，则会作为值设置在数据条目中。该值最长为 64 bytes。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|MANAGE_DATA_NOT_SUPPORTED_YET| -1| 当前网络尚未升级协议。这个失败意味当前网络尚不支持这个操作。 |
|MANAGE_DATA_NAME_NOT_FOUND| -2| 试图删除一个不存在数据条目。如果您将一个数据条目的值设为空，但是这个账户中并不存在该键(Name)便会发送这样的错误。 |
|MANAGE_DATA_LOW_RESERVE| -3| 该账户的 XLM 余额或售出负债不足以使该账户能够添加一个数据条目。每添加一个数据条目都会增加帐户需要持有的最低账户余额。 |
|MANAGE_DATA_INVALID_NAME| -4| 键(Name)是无效的字符串。 |

## Bump Sequence(增大序列号)
[JavaScript](http://stellar.github.io/js-stellar-sdk/Operation.html#.bumpSequence) | [Java](http://stellar.github.io/java-stellar-sdk/org/stellar/sdk/BumpSequenceOperation.Builder.html) | [Go](https://godoc.org/github.com/stellar/go/build#BumpSequenceBuilder)

*仅在协议版本 10 及以上版本中可用*

Bump sequence 操作允许源账户增大自己的序列号，通过该操作您可以使序列号小于新序列号的所有事务失效。

如果指定的 `bumpTo` 序列号大于源账户的序列号，则系统会使用该值更新账户的序列号，否则不会修改。

阈值等级：低

结果：`BumpSequenceResult`

|参数| 类型| 描述|
| --- | --- | --- |
|bumpTo| SequenceNumber| 您想要将操作的源账户的序列号更改为该值。 |

可能发送的错误：

| 错误代码 | 编号 | 描述 |
| ----- | ---- | ------|
|BUMP_SEQUENCE_BAD_SEQ| -1| 提供的 `bumpTo` 序列号无效。该值必须在 0 到 `INT64_MAX` (INT64_MAX 等于 9223372036854775807 或 0x7fffffffffffffff) 之间。 |
