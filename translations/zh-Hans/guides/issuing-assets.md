---
title: 发行资产
---

Stellar 最强大的功能之一便是能够交易任何类型的资产，例如美元、尼日利亚奈拉、比特币、优惠券、[ICO 代币](https://www.stellar.org/blog/tokens-on-stellar/)或是任何您喜欢的东西。

这种功能之所以能在 Stellar 网络中存在，是因为您在 Stellar 网络中持有资产实际上是由发行方发行的信贷。当您在 Stellar 网络上交易美元时，您交易的并不是真正的美元，而是*由特定账户发放的美元信贷*，一般来说，这个账户由银行持有，但如果您的邻居有香蕉农场，他也可以发行香蕉资产。

每种资产(Lumens 除外)都由以下两个属性定义：

- `asset_code`: 由 1 到 12 个字母或数字组成的的资产代码，例如 `USD` 、`EUR`。您也可以将它设置为任何您喜欢的代码，比如 `AstroDollars`。
- `asset_issuer`: 发行资产的账户 ID。

在 Stellar SDK 中，资产使用 `Asset` 类表示：

<code-example name="在 SDK 中表示一种资产">

```js
var astroDollar = new StellarSdk.Asset(
  'AstroDollar', 'GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF');
```

```java
KeyPair issuer = StellarSdk.Keypair.fromAccountId(
  "GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF");
Asset astroDollar = Asset.createNonNativeAsset("AstroDollar", issuer);
```

```json
// 在 Horizon 中，资产使用以下结构的 JSON 表示
{
  "asset_code": "AstroDollar",
  "asset_issuer": "GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF",
  // `asset_type` 用于标识资产数据的存储方式
  // 它可以是 `native` (lumens), `credit_alphanum4`, 或 `credit_alphanum12`.
  "asset_type": "credit_alphanum12"
}
```

</code-example>


## 发行新类型的资产

要发布新类型的资产，您只需选择一个资产代码即可。它可以是 1 到 12 个字母或数字的任意组合，但您应使用适当的 [ISO 4217 代码](https://en.wikipedia.org/wiki/ISO_4217)（例如 `USD` ），或使用 [ISIN 代码](https://en.wikipedia.org/wiki/International_Securities_Identification_Number)表示国家货币或证券。在确定了合适的代码后，您就可以开始使用该资产向用户付款了。您无需执行任何操作即可在网络上声明该资产。

但是，其他人无法收到您的资产，除非他们对您发行的资产抱有信心。因为 Stellar 上的资产实际上是一种信贷，您应该相信发行人可以在必要时兑换该信贷。举例来说，如果您的邻居连一个香蕉都没有，那么您肯定不会愿意信任他发行的香蕉资产。

账户可以通过使用 [changeTrust 操作](concepts/list-of-operations.md#change-trust)创建某个资产的*信任线*来信任它。您可以在信任线中设置自己能持有的该资产的上限，比如您不相信您的邻居拥有超过 200 个香蕉，那么您可以在信任线中将持有上限设置为 200。*注意：每添加一条新的信任线都会增加账户需要持有的最低账户余额（增加 0.5 XLM）。想要了解更多请参阅[费用](concepts/fees.html#minimum-balance)。*

一旦您确定了资产代码并且其他人创建了包含您的资产的信任线，您就可以开始使用您的资产向他们发送付款了。如果您的收款对象不信任您的资产，您也可以使用[分布式交易所](https://stellar-docs.overcat.me/guides/concepts/exchange.md)将自己的资产兑换为他能接收的资产。

### 来试一试吧

发送和接收自定义资产与[发送和接收 Lumens](https://stellar-docs.overcat.me/guides/get-started/transactions.html#building-a-transaction) 非常相似。这是一个简单的示例：

<code-example name="发送自定义资产">

```js
var StellarSdk = require('stellar-sdk');
StellarSdk.Network.useTestNetwork();
var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');

// 发行资产与接收资产的账户的密钥
var issuingKeys = StellarSdk.Keypair
  .fromSecret('SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4');
var receivingKeys = StellarSdk.Keypair
  .fromSecret('SDSAVCRE5JRAI7UFAVLE5IMIZRD6N6WOJUWKY4GFN34LOBEEUS4W2T2D');

// 创建一个用于表示自定义资产的对象
var astroDollar = new StellarSdk.Asset('AstroDollar', issuingKeys.publicKey());

// 首先，我们需要让收款账户信任该资产
server.loadAccount(receivingKeys.publicKey())
  .then(function(receiver) {
    var transaction = new StellarSdk.TransactionBuilder(receiver)
      // `changeTrust` 操作新建(或修改)一条信任线
      // `limit` 参数是可选的，用于设置该账户最多能持有该资产的数目
      .addOperation(StellarSdk.Operation.changeTrust({
        asset: astroDollar,
        limit: '1000'
      }))
      .build();
    transaction.sign(receivingKeys);
    return server.submitTransaction(transaction);
  })

  // 然后，使用发行账户发送给收款账户发送一些资产
  .then(function() {
    return server.loadAccount(issuingKeys.publicKey())
  })
  .then(function(issuer) {
    var transaction = new StellarSdk.TransactionBuilder(issuer)
      .addOperation(StellarSdk.Operation.payment({
        destination: receivingKeys.publicKey(),
        asset: astroDollar,
        amount: '10'
      }))
      .build();
    transaction.sign(issuingKeys);
    return server.submitTransaction(transaction);
  })
  .catch(function(error) {
    console.error('Error!', error);
  });
```

```java
Network.useTestNetwork();
Server server = new Server("https://horizon-testnet.stellar.org");

// 发行资产与接收资产的账户的密钥
KeyPair issuingKeys = KeyPair
  .fromSecretSeed("SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4");
KeyPair receivingKeys = KeyPair
  .fromSecretSeed("SDSAVCRE5JRAI7UFAVLE5IMIZRD6N6WOJUWKY4GFN34LOBEEUS4W2T2D");

// 创建一个用于表示自定义资产的对象
Asset astroDollar = Asset.createNonNativeAsset("AstroDollar", issuingKeys);

// 首先，我们需要让收款账户信任该资产
AccountResponse receiving = server.accounts().account(receivingKeys);
Transaction allowAstroDollars = new Transaction.Builder(receiving)
  .addOperation(
    // `changeTrust` 操作新建(或修改)一条信任线
    // 第二个参数为 `limit`，它是可选的，用于设置该账户最多能持有该资产的数目
    new ChangeTrustOperation.Builder(astroDollar, "1000").build())
  .build();
allowAstroDollars.sign(receivingKeys);
server.submitTransaction(allowAstroDollars);

// 然后，使用发行账户发送给收款账户发送一些资产
AccountResponse issuing = server.accounts().account(issuingKeys);
Transaction sendAstroDollars = new Transaction.Builder(issuing)
  .addOperation(
    new PaymentOperation.Builder(receivingKeys, astroDollar, "10").build())
  .build();
sendAstroDollars.sign(issuingKeys);
server.submitTransaction(sendAstroDollars);
```

```go
issuerSeed := "SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4"
recipientSeed := "SDSAVCRE5JRAI7UFAVLE5IMIZRD6N6WOJUWKY4GFN34LOBEEUS4W2T2D"

// 发行资产与接收资产的账户的密钥
issuer, err := keypair.Parse(issuerSeed)
if err != nil { log.Fatal(err) }
recipient, err := keypair.Parse(recipientSeed)
if err != nil { log.Fatal(err) }

// 创建一个用于表示自定义资产的对象
astroDollar := build.CreditAsset("AstroDollar", issuer.Address())

// 首先，我们需要让收款账户信任该资产
trustTx, err := build.Transaction(
    build.SourceAccount{recipient.Address()},
    build.AutoSequence{SequenceProvider: horizon.DefaultTestNetClient},
    build.TestNetwork,
    build.Trust(astroDollar.Code, astroDollar.Issuer, build.Limit("100.25")),
)
if err != nil { log.Fatal(err) }
trustTxe, err := trustTx.Sign(recipientSeed)
if err != nil { log.Fatal(err) }
trustTxeB64, err := trustTxe.Base64()
if err != nil { log.Fatal(err) }
_, err = horizon.DefaultTestNetClient.SubmitTransaction(trustTxeB64)
if err != nil { log.Fatal(err) }

// 然后，使用发行账户发送给收款账户发送一些资产
paymentTx, err := build.Transaction(
    build.SourceAccount{issuer.Address()},
    build.TestNetwork,
    build.AutoSequence{SequenceProvider: horizon.DefaultTestNetClient},
    build.Payment(
        build.Destination{AddressOrSeed: recipient.Address()},
        build.CreditAmount{"AstroDollar", issuer.Address(), "10"},
    ),
)
if err != nil { log.Fatal(err) }
paymentTxe, err := paymentTx.Sign(issuerSeed)
if err != nil {	log.Fatal(err) }
paymentTxeB64, err := paymentTxe.Base64()
if err != nil { log.Fatal(err) }
_, err = horizon.DefaultTestNetClient.SubmitTransaction(paymentTxeB64)
if err != nil { log.Fatal(err) }
```

</code-example>

## **可发现性和元信息**

在发布资产时，另一件重要的事情是让别人知道您的资产有什么含义。这些信息应该能够被用户找到并显示出来，这样用户就可以准确地知道他们在持有您的资产时得到了什么东西。要做到这一点，您需要做两件简单的事情。首先，在 [stellar.toml 文件](concepts/stellar-toml.html)中添加一个包含必要元字段的部分：

```
# stellar.toml 资产数据示例：
[[CURRENCIES]]
code="GOAT"
issuer="GD5T6IPRNCKFOHQWT264YPKOZAWUMMZOLZBJ6BNQMUGPWGRLBK3U7ZNP"
display_decimals=2
name="goat share"
desc="1 GOAT token entitles you to a share of revenue from Elkins Goat Farm."
conditions="There will only ever be 10,000 GOAT tokens in existence. We will distribute the revenue share annually on Jan. 15th"
image="https://pbs.twimg.com/profile_images/666921221410439168/iriHah4f.jpg"
```

然后使用 [setOptions 操作](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#set-options)将发行账户的 `home_domain` 设置为托管 stellar.toml 文件的域名。以下是设置 `home_domain` 的示例：

<code-example name="设置主域名">

```js
var StellarSdk = require('stellar-sdk');
StellarSdk.Network.useTestNetwork();
var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');

// 发行账户的密钥
var issuingKeys = StellarSdk.Keypair
  .fromSecret('SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4');

server.loadAccount(issuingKeys.publicKey())
  .then(function(issuer) {
    var transaction = new StellarSdk.TransactionBuilder(issuer)
      .addOperation(StellarSdk.Operation.setOptions({
        homeDomain: 'yourdomain.com',
      }))
      .build();
    transaction.sign(issuingKeys);
    return server.submitTransaction(transaction);
  })
  .catch(function(error) {
    console.error('Error!', error);
  });
```

```java
Network.useTestNetwork();
Server server = new Server("https://horizon-testnet.stellar.org");

// 发行账户的密钥
KeyPair issuingKeys = KeyPair
  .fromSecretSeed("SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4");
AccountResponse sourceAccount = server.accounts().account(issuingKeys);

Transaction setHomeDomain = new Transaction.Builder(sourceAccount)
  .addOperation(new SetOptionsOperation.Builder()
    .setHomeDomain("yourdomain.com").build()
  .build();
setAuthorization.sign(issuingKeys);
server.submitTransaction(setHomeDomain);

```

</code-example>


## 将资产设置为需要授权并可以被冻结

账户有几个与发行资产相关的[标识](concepts/accounts.md#flags)。

如果您想控制谁能够接收您发行的资产，或者出于某些特殊目的，您希望用户在使用该资产时需要得到您的授权，您可以通过启用 [`AUTHORIZATION REQUIRED` 标识](https://www.stellar.org/developers/guides/concepts/assets.html#controlling-asset-holders)来获得这样的权利，当该标识为被启用后，接收账户在获得批准后才能接收该资产。

[`AUTHORIZATION REVOCABLE` 标识](concepts/assets.md#controlling-asset-holders)允许您在发生盗窃或其他特殊情况时冻结您发放的资产。这可能对国家货币有用，但并不总是适用于其他类型的资产。

在大多数情况下，您不应该启用 `AUTHORIZATION REVOCABLE` 标识。在过去，一些发行人使用 `AUTHORIZATION REVOCABLE` 标志来强制执行锁定期。但是，这样做会产生一个问题，因为它不会向用户提供有关资产何时能解锁或是能否解锁的任何保证。

更重要的是，*一旦设置完成就需要花费很多精力才能撤消这个操作带来的影响*。这是因为即使您后来关闭了 `AUTHORIZATION REVOCABLE` 或 `AUTHORIZATION REQUIRED` 标识，也不会对之前已经存在的账户造成任何影响。为了让之前已经存在的账户能够收到该资产，您需要对这些账户执行 [`Allow Trust`](concepts/list-of-operations.md#allow-trust) 操作。
您需要为*每个*已存在的用户帐户创建一个具有以下操作的事务：

1. 使用 [`Set Options`](concepts/list-of-operations.md#set-options) 操作将 `AUTHORIZATION REQUIRED` 标识设置为 `0x1`。这个操作是必要的，因为这个标识没有启用的话， [`Allow Trust`](concepts/list-of-operations.md#allow-trust) 操作将无法执行。
2. 使用 [`Allow Trust`](concepts/list-of-operations.md#allow-trust) 操作对已存在的所有的账户进行授权。**注意：**您可以最多在这里放置 `MAX OPERATIONS PER
   TRANSACTION - 2` ([MAX OPERATIONS PER
   TRANSACTION 目前为 100](concepts/transactions.md#list-of-operations)) 个
   `Allow Trust` 操作。
3. 使用 [`Set Options`](concepts/list-of-operations.md#set-options) 操作将账户的 `AUTHORIZATION REQUIRED` 标识设置为 `0x0`。

如果希望通过将 `AUTHORIZATION REQUIRED` 标识设置为 `0x0` 来使后续用户使用资产而不需要经过发行账户授权，那么**提交这些事务就是必须的**。

以下示例会将资产设置为需要授权并可以被冻结：

<code-example name="资产授权">

```js
StellarSdk.Network.useTestNetwork();
var transaction = new StellarSdk.TransactionBuilder(issuingAccount)
  .addOperation(StellarSdk.Operation.setOptions({
    setFlags: StellarSdk.AuthRevocableFlag | StellarSdk.AuthRequiredFlag
  }))
  .build();
transaction.sign(issuingKeys);
server.submitTransaction(transaction);
```

```java
import org.stellar.sdk.AccountFlag;

Network.useTestNetwork();
Server server = new Server("https://horizon-testnet.stellar.org");

// 发行账户的密钥
KeyPair issuingKeys = KeyPair
  .fromSecretSeed("SCZANGBA5YHTNYVVV4C3U252E2B6P6F5T3U6MM63WBSBZATAQI3EBTQ4");
AccountResponse sourceAccount = server.accounts().account(issuingKeys);

Transaction setAuthorization = new Transaction.Builder(sourceAccount)
  .addOperation(new SetOptionsOperation.Builder()
    .setSetFlags(
      AccountFlag.AUTH_REQUIRED_FLAG.getValue() |
      AccountFlag.AUTH_REVOCABLE_FLAG.getValue())
    .build())
  .build();
setAuthorization.sign(issuingKeys);
server.submitTransaction(setAuthorization);
```

</code-example>

## 最佳实践

当您决定发布自己的资产后，这里有一些最佳实践可以帮助您提高安全性并简化管理。


### 将所有的操作放在一个事务中提交

发行资产时，Stellar 网络上的所有操作（创建帐户，设置账户选项，付款等）都应该放在一个事务中提交。因为 Stellar 中的事务具有原子性，这意味着所有操作要么都成功，要么都失败。这可以确保资产发行不会陷入中间状态，例如，账户创建成功但是没有成功接收到资产。


### 独立的资产发行账户

在最简单的情况下，您可以将您日常使用的账户用于发行资产。但是，如果您经营着一家金融机构或企业，您应该使用一个专门的账户专门发行资产。为什么？

- 容易追踪：因为资产代表信贷，当它被发送回发行账户时，它就消失了。为了更好地跟踪和控制流通中的资产数额，您可以利用发行账户向一般的工作账户支付固定数额的资产，而后续工作都可通过工作账户进行。

  发行账户可以根据手中持有的实际资产（比如美元或香蕉）发行更多的信贷，而工作账户不必关心到底有多少资产被发行了。

- 使授信操作保持简单：随着您扩大对 Stellar 的使用，您可能会出于各种原因而考虑拥有多个账户，比如说您想以高速率向网络提交事务。使用一个独立且规范的发行账户可以让其他人更轻松地了解他们将要信任的账户。


### 在付款前检查是否被信任

因为每笔事务都要收取一小笔费用，所以您可能需要对收款账户的信任线进行检查，以确保收款账户能够顺利接收您的付款。如果这个账户设置了包含您发行资产的信任线，那么它将会被列在帐户`余额`中(即使余额是 `0`)。

<code-example name="检查信任线">

```js
var astroDollarCode = 'AstroDollar';
var astroDollarIssuer =
  'GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF';

var accountId = 'GA2C5RFPE6GCKMY3US5PAB6UZLKIGSPIUKSLRB6Q723BM2OARMDUYEJ5';
server.loadAccount(accountId).then(function(account) {
  var trusted = account.balances.some(function(balance) {
    return balance.asset_code === astroDollarCode &&
           balance.asset_issuer === astroDollarIssuer;
  });

  console.log(trusted ? 'Trusted :)' : 'Not trusted :(');
});
```

```java
Asset astroDollar = Asset.createNonNativeAsset(
  "AstroDollar",
  KeyPair.fromAccountId(
    "GC2BKLYOOYPDEFJKLKY6FNNRQMGFLVHJKQRGNSSRRGSMPGF32LHCQVGF"));

// 载入您想检查的账户
KeyPair keysToCheck = KeyPair.fromAccountId(
  "GA2C5RFPE6GCKMY3US5PAB6UZLKIGSPIUKSLRB6Q723BM2OARMDUYEJ5");
AccountResponse accountToCheck = server.accounts().account(keysToCheck);

// 查看余额中是否包含了我们发行资产的信任线
boolean trusted = false;
for (AccountResponse.Balance balance : accountToCheck.getBalances()) {
  if (balance.getAsset().equals(astroDollar)) {
    trusted = true;
    break;
  }
}

System.out.println(trusted ? "Trusted :)" : "Not trusted :(");
```

</code-example>


## **为您的资产提供流动性**

当有人试图在分布式交易所买卖您的资产时，他需要有能与他进行交易的交易对手。也就是说，您的资产需要有足够的流动性。

这个问题可以由[做市商][market-maker]解决。做市商可以是签约提供此服务的人，也可以是资产发行人，您还可以使用[做市机器人][kelp]来为您的资产提供流动性。


## 其它与资产相关的信息

现在您已经对自定义资产有了基本的了解，请通过[资产概念文档](https://stellar-docs.overcat.me/guides/concepts/assets.html)了解更多的技术细节。


[ISO 4217]: https://en.wikipedia.org/wiki/ISO_4217
[ISIN]: https://en.wikipedia.org/wiki/International_Securities_Identification_Number
[liquidity]: https://en.wikipedia.org/wiki/Market_liquidity
[market-maker]: https://en.wikipedia.org/wiki/Market_maker
[kelp]: https://github.com/stellar/kelp
