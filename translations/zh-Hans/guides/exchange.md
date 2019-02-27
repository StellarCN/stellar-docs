---
title: 将 Stellar 添加到您的交易所中
---

本文将向您介绍如何将 Stellar 网络中的令牌添加到您的交易所中。首先，我们将介绍 Stellar 的原生资产 Lumens，随后，我们将介绍其它类型的令牌。此示例使用 Node.js 和 [JS Stellar SDK](https://github.com/stellar/js-stellar-sdk)，但您能很容易地使用其它语言来实现这些功能。

交易所能采用的设计有很多种。本指南使用以下设计：
 - `issuing account(发行账户)`: 用于储存大部分用户存款的离线账户。
 - `base account(基础账户)`: 用于处理提现的在线账户，这个账户中持有少量的用户存款。
 - `customerID(客户 ID)`: 每个用户都有一个 customerID，用于将收到的存款与交易所中特定用户的账户相关联。

交易所想要集成 Stellar 的话，需要实现这两点：<br>
1) 从 Stellar 网络监听存款事务<br>
2) 将提现事务提交到 Stellar 网络

## 设置

### 可选
* *(可选)* 设置 [Stellar Core](https://www.stellar.org/developers/stellar-core/software/admin.html)
* *(可选)* 设置 [Horizon](https://www.stellar.org/developers/horizon/reference/index.html)

上述操作虽然不是必须的，但我们建议您运行自己的 Stellar Core 和 Horizo​​n 实例 —— [这篇文章](https://www.stellar.org/developers/stellar-core/software/admin.html#why-run-a-node)列出了这样做的诸多好处。如果您实在不想这样做的话，可以使用 Stellar.org 运行的面向公众的 Horizo​​n 服务器。我们的测试和公共网络如下：

```
  测试网络： {hostname:'horizon-testnet.stellar.org', secure:true, port:443};
  公共网络： {hostname:'horizon.stellar.org', secure:true, port:443};
```

### 发行账户
发行账户通常用于保护大量资金的安全。发行账户是 Stellar 账户，其密钥不储存在任何联网的设备上。事务由用户构建并在离线的计算机上进行签名 —— 您可以在离线计算机上安装 `js-stellar-sdk` ，然后使用它创建包含签名的事务 `tx_blob`。该 `tx_blob` 可以通过离线方式（例如，USB 或手动）传输到已联网的机器。这种设计使得发行账户的密钥更加安全。

### 基础账户
基础账户包含的资金数量比发行账户的数量要少得多。基本账户是在联网的计算机上使用的 Stellar 账户。它处理 Lumens 的日常发送和接收任务。基本账户中的有限资金能够减少发生安全漏洞时所产生的损失。

### 数据库
- 需要为待处理的提现创建一张表 `StellarTransactions`。
- 需要创建一张表 `StellarCursor`，保存用于监听存款信息的最新游标。
- 需要在 `Users` 表中为每个用户添加 `customerID` 行。
- 需要在 `customerID` 行填写合适的信息。

```
CREATE TABLE StellarTransactions (UserID INT, Destination varchar(56), XLMAmount INT, state varchar(8));
CREATE TABLE StellarCursor (id INT, cursor varchar(50)); // id - AUTO_INCREMENT field
```

`StellarTransactions.state` 可能的值为 "pending(待处理)", "done(处理完成)", "error(错误)"。


### 代码

使用此代码框架将 Stellar 集成到您的交易所中。对于本指南，我们使用占位符函数来读取/写入交易所的数据库。每个数据库的连接方式不同，因此我们抽象出这些细节。以下部分描述了每个步骤：


```js
// 配置您的服务器
var config = {};
config.baseAccount = "your base account address";
config.baseAccountSecret = "your base account secret key";

// 您可以使用 Stellar.org 提供的 Horizon 节点，也可以使用您自己搭建的节点
config.horizon = 'https://horizon-testnet.stellar.org';

// 导入 js-stellar-sdk 库，通过它，客户端可以与 Horizon 通讯
var StellarSdk = require('stellar-sdk');
// 如果想使用公共网络的话，请不要注释下面这行
// StellarSdk.Network.usePublicNetwork();

// 使用您的 Horizon 配置初始化 SDK
var server = new StellarSdk.Server(config.horizon);

// 获取最新的游标位置
var lastToken = latestFromDB("StellarCursor");

// 从您上次停下的地方开始监听充值信息。
// GET https://horizon-testnet.stellar.org/accounts/{config.baseAccount}/payments?cursor={last_token}
let callBuilder = server.payments().forAccount(config.baseAccount);

// 如果还没有保存游标参数的话，就不要添加它
if (lastToken) {
  callBuilder.cursor(lastToken);
}

callBuilder.stream({onmessage: handlePaymentResponse});

// 从 Horizon 获取账户的序列号，并返回该账户
// GET https://horizon-testnet.stellar.org/accounts/{config.baseAccount}
server.loadAccount(config.baseAccount)
  .then(function (account) {
    submitPendingTransactions(account);
  })
```

## 监听充值信息
当用户想将 Lumens 存入您的交易所时，请指导他们将 XLM 发送到您的基本账户中，并在事务的备忘录(Memo)中填写 customerID。

您必须监听基本账户的收款信息，并将收到的 XLM 记录在用户账上。这是监听收款信息的代码：

```js
// 从您上次停下的地方开始监听充值信息。
var lastToken = latestFromDB("StellarCursor");

// GET https://horizon-testnet.stellar.org/accounts/{config.baseAccount}/payments?cursor={last_token}
let callBuilder = server.payments().forAccount(config.baseAccount);

// 如果还没有保存游标参数的话，就不要添加它
if (lastToken) {
  callBuilder.cursor(lastToken);
}

callBuilder.stream({onmessage: handlePaymentResponse});
```


对于基本账户收到的每笔付款，您都需要做以下事情：<br>
 - 检查备忘录(Memo)字段以确定这是由哪个用户发送的存款。<br>
 - 将游标记录在 `StellarCursor` 表中，以便您可以从中断处继续进行存款业务。
 - 将收到的 XLM 记录在用户的账上。

所以，当您在以 Stream 的方式监听用户付款时，将以下函数传递给 `onmessage`：

```js
function handlePaymentResponse(record) {

  // GET https://horizon-testnet.stellar.org/transaction/{此付款的事务 ID}
  record.transaction()
    .then(function(txn) {
      var customer = txn.memo;

      // 如果不是付款给基本账户则忽略
      if (record.to != config.baseAccount) {
        return;
      }
      if (record.asset_type != 'native') {
         // 如果您的交易所是一家 XLM 交易所，而用户给您发送了其它类型的资产，
         // 您可以有以下两种处理方式：
         // 1. 将它兑换为原生资产然后记录在用户账上
         // 2. 将它退回给用户
      } else {
        // 根据 Memo 将存款记录在用户账上
        if (checkExists(customer, "ExchangeUsers")) {
          // 更新数据库，该操作具有原子性
          db.transaction(function() {
            // 将用户存款记录在他们的账上
            store([record.amount, customer], "StellarDeposits");
            // 将游标记录在数据库中
            store(record.paging_token, "StellarCursor");
          });
        } else {
          // 如果这个用户不存在，您可以抛出一个错误信息，
          // 也可以把他们添加到您的用户列表中，然后将付款记录在他们账上，
          // 当然您也可以按照自己的想法进行处理
          console.log(customer);
        }
      }
    })
    .catch(function(err) {
      // 错误处理
    });
}
```


## 处理提现
当用户申请对 XLM 进行提现时，您需要构建一个事务来发送 XLM。请参阅[构建事务](https://www.stellar.org/developers/js-stellar-base/learn/building-transactions.html)以了解更多信息。

每当有用户申请提现时，`handleRequestWithdrawal` 函数将构建一个事务，这些事务会保存在交易所的 `StellarTransactions` 表中，然后被依次执行。

```js
function handleRequestWithdrawal(userID,amountLumens,destinationAddress) {
  // 更新数据库，该操作具有原子性
  db.transaction(function() {
    // 从数据库中读取用户的余额
    var userBalance = getBalance('userID');

    // 检查用户是否有足够的 XLM
    if (amountLumens <= userBalance) {
      // 根据用户提现的金额记录用户的余额
      store([userID, userBalance - amountLumens], "UserBalances");
      // 将事务信息保存在 `StellarTransactions` 表中
      store([userID, destinationAddress, amountLumens, "pending"], "StellarTransactions");
    } else {
      // 如果用户没有足够的 XLM 的话，您可以提醒他们
    }
  });
}
```

随后，您应该执行 `submitPendingTransactions`，它会检查等待提交事务的 `StellarTransactions`，然后提交它们。

```js
StellarSdk.Network.useTestNetwork();
// 这个函数可以提交一个事务

function submitTransaction(exchangeAccount, destinationAddress, amountLumens) {
  // 将事务状态更新为正在发送，以便在发生错误时不会重新提交。
  updateRecord('sending', "StellarTransactions");

  // 检查收款地址是否存在
  // GET https://horizon-testnet.stellar.org/accounts/{destinationAddress}
  server.loadAccount(destinationAddress)
    // 如果操作的话，向收款地址付款。
    .then(function(account) {
      var transaction = new StellarSdk.TransactionBuilder(exchangeAccount)
        .addOperation(StellarSdk.Operation.payment({
          destination: destinationAddress,
          asset: StellarSdk.Asset.native(),
          amount: amountLumens
        }))
        // 签署该事务
        .build();

      transaction.sign(StellarSdk.Keypair.fromSecret(config.baseAccountSecret));

      // POST https://horizon-testnet.stellar.org/transactions
      return server.submitTransaction(transaction);
    })
    // 如果收款账户不存在的话... ...
    .catch(StellarSdk.NotFoundError, function(err) {
      // 在网络中创建这个账户，初始金额设置为用户的提款金额
      var transaction = new StellarSdk.TransactionBuilder(exchangeAccount)
        .addOperation(StellarSdk.Operation.createAccount({
          destination: destinationAddress,
          // 您需要给它发送一定的初始金额以在网络中创建该账户
          startingBalance: amountLumens
        }))
        .build();

      transaction.sign(StellarSdk.Keypair.fromSecret(config.baseAccountSecret));

      // POST https://horizon-testnet.stellar.org/transactions
      return server.submitTransaction(transaction);
    })
    // 提交事务
    .then(function(transactionResult) {
      updateRecord('done', "StellarTransactions");
    })
    .catch(function(err) {
      // 捕获错误，最有可能是网络错误或是事务执行失败
      updateRecord('error', "StellarTransactions");
    });
}

// 这个函数会提交所有等待的事务，并不断调用自己
// 此函数应该在后台持续运行

function submitPendingTransactions(exchangeAccount) {
  // 查看数据库中有那些事务等待处理
  // 更新数据库，该操作具有原子性
  db.transaction(function() {
    var pendingTransactions = querySQL("SELECT * FROM StellarTransactions WHERE state =`pending`");

    while (pendingTransactions.length > 0) {
      var txn = pendingTransactions.pop();

      // 这个函数使用了 async，所以它不会阻塞。
      // 为简单起见，我们使用的是 ES7 中的 `await`关键字，
      // 但您应该创建一个 "promise 流"，以便在提交所有事务后执行下面的 "setTimeout" 行。
      // 如果您不这样做，则可能会提交两次或更多次事务。
      await submitTransaction(exchangeAccount, tx.destinationAddress, tx.amountLumens);
    }

    // 等待 30 秒后处理下一系列的事务。
    setTimeout(function() {
      submitPendingTransactions(sourceAccount);
    }, 30*1000);
  });
}
```

## 更进一步... ...
### 联邦服务
联邦协议允许您为用户提供简单的地址(例如 bob*yourexchange.com)，而不是繁琐的原始地址(例如 GCEZWKCA5VLDNRLN3RPRJMRZOX3Z6G5CHCGSNFHEYVXM3XOJMDS674JZ?19327)。

有关更多信息，请查阅[联邦服务指南](./concepts/federation.md)。

### 锚点
如果您是一个交易所的话，那么很容易成为一个恒星锚点。锚点是人们信任的实体，用于持有存款并在 Stellar 网络中发放信贷。因此，它们在现有货币和 Stellar 网络之间起桥梁作用。成为锚点能够扩展您的业务。

想要更多地了解成为一个锚点的意义，请参阅[锚点指南](./anchor)。

### 接收其它类型资产
如果您想接受其它类型的令牌，请按照这些说明操作。

您需要为发行账户创建一条包含了您想接收的令牌的[信任线](https://www.stellar.org/developers/guides/concepts/assets.html#trustlines)，除非您创建了该信任线，否则您无法接收该类型的资产。

```js
var someAsset = new StellarSdk.Asset('ASSET_CODE', issuingKeys.publicKey());

transaction.addOperation(StellarSdk.Operation.changeTrust({
        asset: someAsset
}))
```
如果该资产的发行方启用了 `authorization_required` 标识，在他授权您接收此资产之前，您都无法收取该资产。请阅读[这篇文章](https://www.stellar.org/developers/guides/concepts/assets.html#controlling-asset-holders)来了解这意味着什么。

然后，对上面的示例代码进行一些更改：
* 在 `handlePaymentResponse` 函数中，我们处理了用户发送非原生资产的情况。由于我们现在可以接受其它类型的令牌，您需要修改此条件，如果用户向我们发送 XLM，我们将可以：
	1. 将 Lumens 兑换为其它类型的目标资产
	2. 将 Lumens 发回给用户

*注意*：如果发行账户没有创建包含某种令牌的信任线，那么用户将无法向我们发送这种资产。

* 在 `withdraw` 函数中，在我们向用户发送资产时，我们必须指定发送的是哪种令牌。示例代码：
```js
var someAsset = new StellarSdk.Asset('ASSET_CODE', issuingKeys.publicKey());

transaction.addOperation(StellarSdk.Operation.payment({
        destination: receivingKeys.publicKey(),
        asset: someAsset,
        amount: '10'
      }))
```
* 在 `withdraw` 函数中，您的用户必须为他提取的令牌创建了相应的信任线。所以您必须考虑以下因素：
	* 确认接收令牌的用户具有相应的信任线
	* 将令牌发送到没有相应的信任线的账户后将发生的[Horizon 错误](https://www.stellar.org/developers/guides/concepts/list-of-operations.html#payment)


如果想对资产有着更多的了解，请阅读这两篇文章：[资产](https://www.stellar.org/developers/guides/concepts/assets.html)和[发行资产指南](https://www.stellar.org/developers/guides/issuing-assets.html)。
