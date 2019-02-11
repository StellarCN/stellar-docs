---
title: Stellar 附件公约
---

# 附件

有时您需要在事务中提供一些其它的信息，但是这些信息无法存入备注(Memo)中，例如：KYC 信息、发票、简短说明。由于它的大小或私人性质，这些数据不应该放在[总账](./concepts/ledger.md)中。您应该创建一个`附件`，Stellar 附件是一个 JSON 文档。此附件的 sha256 hash 作为备注(Memo)被包含在事务中。实际的附件文档可以通过其他途径发送给接收方，例如：通过接收方的[认证服务器](./compliance-protocol.md)。

## 附件的结构

附件具有灵活的结构。它们可以选择性地包含以下字段，并且可以附加额外的信息。

```json
{
  "nonce": "<nonce>",
  "transaction": {
    "sender_info": {
      "first_name": "<first_name>",
      "middle_name": "<middle_name>",
      "last_name": "<last_name>",
      "address": "<address>",
      "city": "<city>",
      "province": "<province>",
      "country": "<country in ISO 3166-1 alpha-2 format>",
      "date_of_birth": "<date of birth in YYYY-MM-DD format>",
      "company_name": "<company_name>"
    },
    "route": "<route>",
    "note": "<note>"
  },
  "operations": [
    {
      "sender_info": <sender_info>,
      "route": "<route>",
      "note": "<note>"
    },
    // ...
  ]
}
```

字段名 | 数据类型 | 描述 
-----|-----------|------------
`nonce` | string | [Nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) 是一个独一无二的值。您发送的每个事务都应该有着不同的 Nonce 值。系统需要通过 nonce 来区分两个带有相同细节的事务的附件。例如，您连续两天寄 10 美元给鲍勃。 
`transaction.sender_info` | JSON | 这个 JSON 中包含了发送者的 KYC 信息。如果需要的话，这个 JSON 中可以包含更多的信息。 
`transaction.route` | string | 路径信息由联邦服务器提供(`memo` 值)。告诉接收方如何将事务发送给最终的接收方。 
`transaction.note` | string | 备注信息。 
`operations[i]` | | 第 `i` 个操作的数据。如果这个事务只有一个操作的话可以省略。 
`operations[i].sender_info` | JSON | 第 `i` 个操作的 `sender_info`。如果为空的话则会从事务中继承该值。 
`operations[i].route` | string | 第 `i`个操作的 `route`。如果为空的话则会从事务中继承该值。 
`operations[i].note` | string | 第 `i`个操作的 `note`。如果为空的话则会从事务中继承该值。 

## 计算附件的 hash

要计算附件的 hash，需要对 JSON 对象进行字符串化并计算其 `sha-256` hash。在 Node.js 中：

```js
const crypto = require('crypto');
const hash = crypto.createHash('sha256');

hash.update(JSON.stringify(attachment));
var memoHashHex = hash.digest('hex');
```

您可以使用 [`TransactionBuilder.addMemo`](http://stellar.github.io/js-stellar-base/TransactionBuilder.html#addMemo) 方法向事务中添加该值。

## 发送附件

若要向接收方的认证服务器发送附件及事务中的散列，请阅读[合规协议](./compliance-protocol.md)的文档以获取更多信息。

## 示例

```js
var crypto = require('crypto');

var nonce = crypto.randomBytes(16);
var attachment = {
  "nonce": nonce.toString('hex'),
  "transaction": {
    "sender_info": {
      "name": "Sherlock Holmes",
      "address": "221B Baker Street",
      "city": "London NW1 6XE",
      "country": "UK",
      "date_of_birth": "1854-01-06"
    }
  },
  "operations": [
    // Operation #1: Payment for Dr. Watson
    {
      "route": "watson",
      "note": "Payment for helping to solve murder case"
    },
    // Operation #2: Payment for Mrs. Hudson
    {
      "route": "hudson",
      "note": "Rent"
    }
  ]
};

var hash = crypto.createHash('sha256');
hash.update(JSON.stringify(attachment));
var memoHashHex = hash.digest('hex');
console.log(memoHashHex);
```
